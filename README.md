# OpenStack Private Cloud Infrastructure

Three OpenStack deployments, built in stages: a single-node DevStack tenant with a default-deny firewall, a two-node Kolla-Ansible cloud provisioned with Heat, and Octavia load balancing across three backends.

**Write-up:** [Building an OpenStack private cloud, and everything that broke](https://cyconic.com/projects/openstack-private-cloud/) walks through the commands, the failures and the fixes.

## Stack

| Area | Tools |
|------|-------|
| Deployment | DevStack (`stable/2023.2`), Kolla-Ansible |
| Hosts | Ubuntu Server 24.04 on VMware ESXi; Nova using QEMU emulation (nested virtualisation) |
| Networking | Neutron, Open vSwitch / OVN, floating IPs |
| Orchestration | Heat (HOT templates) |
| Load balancing | Octavia (HAProxy amphorae) |

## Phases

### 1. Isolated tenant on DevStack

- Tenant network `192.168.100.0/24` with its own router to the external network.
- Security group allowing only SSH (22) and HTTP (80) inbound; everything else denied.
- Custom flavor (1 vCPU, 1 GB RAM, 5 GB disk) to cap resource use per instance.
- Two Ubuntu instances behind floating IPs, verified from outside with `curl` and `nmap -Pn -p 22,80`.

### 2. Two-node Kolla-Ansible cloud with Heat

- Control/network node (`ubuntu-node1`) and compute node (`ubuntu-node2`), all services in Docker containers.
- A Heat template creates the network, subnet, security group and a CirrOS instance in one `openstack stack create`. The template is shown in the Phase 2 report.
- Nova switched to `qemu` because KVM isn't available inside nested VMs.

### 3. Octavia load balancing

- Load balancer `prod-web-lb` with an HTTP listener on port 80 and a round-robin pool of three CirrOS web servers (BusyBox `httpd`).
- Verified from the tenant network's DHCP namespace: six requests rotated evenly across all three backends.

## Troubleshooting summary

| Symptom | Cause | Fix |
|---------|-------|-----|
| DevStack fails partway, disk full | Root LV only uses part of the disk | `lvextend -l +100%FREE` then `resize2fs` |
| DevStack networking dead after reboot | OVS/OVN state not restored | Restart `openvswitch-switch` and `ovn-central`, then `unstack.sh` / `stack.sh` |
| `openstack stack create` not found | Heat disabled; client missing | `enable_heat: "yes"`, `kolla-ansible deploy --tags heat`, `pip install python-heatclient` |
| `NoValidHost` | No KVM inside nested VMs | `nova_compute_virt_type: "qemu"`, `kolla-ansible reconfigure --tags nova` |
| Load balancer stuck in `PENDING_CREATE` / `PENDING_DELETE` | Octavia jobboard enabled without Redis | `enable_octavia_jobboard: "no"`; set stuck rows to `ERROR` in MariaDB, then `delete --cascade` |
| `ComputeBuildException` on amphora | Amphora image owned by `admin` project | `openstack image set --project <service-project-id> <image>` |
| Intermittent 503s | Single-connection backend (Netcat) | Use BusyBox `httpd` |
| Requests to backends time out | No HTTP rule in the security group | Add TCP 80 ingress |

## Repository contents

| File | Covers |
|------|--------|
| `MiniProject1-Openstack-Devstack.pdf` | Phase 1: DevStack tenant, security groups, flavors, floating IPs, full command history |
| `MiniProject2-Openstack-Kolla-Ansible.pdf` | Phase 2: multi-node Kolla-Ansible, Heat template, QEMU fix, full command history |
| `MiniProject3-Openstack-Octavia.pdf` | Phase 3: Octavia deployment, control-plane fixes, round-robin verification, full command history |

## Known limitations

- SSH was opened to any source address; it should be limited to an admin network or bastion host.
- Everything is plain HTTP; TLS termination at the listener (with Barbican) is the next step.
- The load balancer is created by hand rather than in the Heat template.
