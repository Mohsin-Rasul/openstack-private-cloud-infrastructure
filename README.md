# OpenStack Private Cloud Infrastructure

This repository contains the architecture documentation, configuration files, and orchestration templates for deploying and managing a highly secure, multi-node OpenStack private cloud. The infrastructure was built progressively across three distinct phases, moving from a single-node sandbox to a production-grade, container-backed environment with automated load balancing.

## Technical Stack
* **Cloud Platform:** OpenStack (DevStack & Kolla-Ansible)
* **Operating System:** Ubuntu Linux Server (24.04 LTS)
* **Containerization:** Docker
* **Infrastructure as Code (IaC):** OpenStack Heat Orchestration Templates (HOT)
* **Configuration Management:** Ansible
* **Load Balancing:** Octavia LBaaS, HAProxy
* **Networking:** Neutron, Open vSwitch (OVS), Floating IPs
* **Hypervisor:** VMware ESXi, QEMU

## Repository Contents
* `MiniProject1-Openstack-Devstack.pdf`: Documentation on DevStack tenant provisioning and network segmentation.
* `MiniProject2-Openstack-Kolla-Ansible.pdf`: Architecture report for the multi-node containerized deployment.
* `MiniProject3-Openstack-Octavia.pdf`: Diagnostics and configuration guide for the LBaaS traffic routing.
* `heatStack.yaml`: Declarative orchestration template for automating network, subnet, security group, and instance provisioning.

---

## Core Deployment Phases

### Phase 1: Secure Tenant Deployment & Microsegmentation
The foundation of the cloud environment focused on establishing an isolated, access-controlled tenant infrastructure using DevStack. 
* Expanded physical host storage and resolved virtual networking hangs by managing Open vSwitch and OVN services.
* Enforced a zero-trust, default-deny microsegmentation model using security groups at the virtual network interface (vNIC) layer, permitting only explicit SSH and HTTP traffic.
* Decoupled external reachability through 1-to-1 Network Address Translation (NAT) via allocated floating IPs.
* Implemented custom compute flavors to strictly constrain resource allocation and prevent noisy-neighbor performance degradation.

### Phase 2: Multi-Node Kolla-Ansible Cloud & Heat Orchestration
Transitioned the infrastructure from a single-node sandbox to a persistent, highly available multi-node environment (Control Node and Compute Node) using Kolla-Ansible.
* Deployed OpenStack microservices across segregated nodes managed via Docker containers to ensure configuration persistence.
* Overcame nested virtualization limitations by tuning the Nova compute driver to utilize QEMU software emulation.
* Automated the entire infrastructure pipeline—including tenant networks, subnets, and instances—using a declarative Heat Orchestration Template (`heatStack.yaml`), achieving zero-touch provisioning.

### Phase 3: Octavia LBaaS Deployment & Control Plane Diagnostics
Engineered a dynamic traffic management layer using OpenStack Octavia to deploy virtualized HAProxy appliances (Amphorae) for frontend routing.
* Executed deep-system control plane diagnostics to resolve silent API stalls, including MariaDB database state resets and RabbitMQ jobboard queue reconfigurations.
* Corrected Role-Based Access Control (RBAC) constraints by realigning Glance image ownership to the proper service project.
* Validated high availability by deploying persistent lightweight HTTP daemons across a backend compute pool and verifying flawless Round-Robin traffic distribution via the Neutron DHCP network namespace.
