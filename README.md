# OpenStack Private Cloud: Secure Tenant Deployment & Microsegmentation

## Overview
This repository documents the architecture, deployment, and validation of a highly secure, isolated tenant environment within an enterprise-grade OpenStack private cloud. The project demonstrates practical cloud infrastructure engineering by constructing a software-defined network strictly partitioned from the public internet to ensure data sovereignty.

## Security & Architecture Highlights
* **Microsegmentation:** Enforced a zero-trust, default-deny security group policy at the virtual network interface (vNIC) layer, strictly permitting only necessary SSH and HTTP traffic.
* **Network Isolation:** Built an isolated tenant network and subnet, decoupling external reachability through 1-to-1 Network Address Translation (NAT) via floating IPs.
* **Resource Constraint:** Implemented custom compute flavors to cap hardware allocation (RAM, CPU, Disk) and prevent noisy-neighbor performance degradation.

## Technologies Used
* OpenStack (DevStack)
* Localized VMware ESXi Hypervisor
* Open vSwitch & OVN
* Ubuntu Server 24.04 LTS

## Repository Contents
* `OpenStack_Report.pdf`: Comprehensive technical report detailing the deployment methodology, architectural logic, and visual validation.
* `main.tex`: The LaTeX source code for the technical report.
* `commands_history.txt`: The complete bash CLI command history for full reproducibility.
