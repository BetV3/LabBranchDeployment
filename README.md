# Multi-Campus Network Simulation Project

This project simulates a multi-campus network infrastructure in a homelab environment. The design models realistic enterprise scenarios by separating WAN and LAN addressing for each campus and subdividing the LAN into various internal segments for services such as virtualization, domain controllers, file servers, monitoring, backup, and NSX Edge appliances.

## Table of Contents

- [Project Overview](#project-overview)
- [Network Architecture](#network-architecture)
- [Addressing Scheme](#addressing-scheme)
  - [WAN Subnets (Per Campus)](#wan-subnets-per-campus)
  - [LAN Blocks (Per Campus)](#lan-blocks-per-campus)
- [Campus Configuration Details](#campus-configuration-details)
  - [Campus 1 Example](#campus-1-example)
- [Repository Structure](#repository-structure)
- [Automation and Templates](#automation-and-templates)
- [Deployment and Testing](#deployment-and-testing)
- [Future Enhancements](#future-enhancements)
- [References](#references)

## Project Overview

The objective of this project is to deploy a simulated multi-campus network environment with:

- **WAN Connectivity:** Each campus is allocated a dedicated /24 subnet for WAN connectivity. These subnets are used for external routing and redundancy (with VRRP or a similar protocol).
- **Internal LAN:** Each campus receives a private /16 LAN block that is subdivided into smaller /24 segments for different services (e.g., core infrastructure, backup, monitoring, and edge appliances).

Each campus largely replicates the same configuration template, differing only in naming and IP addressing.

## Network Architecture

The overall architecture consists of:

1. **Main OPNsense Router:**  
   - This central router connects to the ISP and allocates WAN subnets to each campus via VLAN trunking.
2. **Campus Routers (Cisco CSR 1000v):**  
   - Each campus uses two routers (redundant pair) configured with VRRP on the WAN interface to ensure high availability.
3. **LAN Infrastructure:**  
   - Each campus receives a large LAN block, which is subdivided for various internal functions.

A simplified diagram for one campus (Campus 1) is shown below:

```
       [ ISP ]
         │
         ▼
[Main OPNsense Router]
         │
         ▼
+---------------------------+
| Campus 1 WAN Subnet       | 192.168.0.0/24  
| VRRP VIP: 192.168.0.1      |
+---------------------------+
         │
  +-------------+   +-------------+
  | Router 1    |   | Router 2    |  (Cisco CSR 1000v, redundant)
  | 192.168.0.2 |   | 192.168.0.3 |  VRRP VIP: 192.168.0.1
  +-------------+   +-------------+
         │
         ▼
+--------------------------------------+
| Campus 1 LAN Block                   | 172.16.0.0/16
| (Subdivided into multiple /24 networks) |
+--------------------------------------+
```

## Addressing Scheme

### WAN Subnets (Per Campus)

Each campus receives a dedicated WAN /24:

| Campus    | WAN Subnet       | WAN VIP      |
|-----------|------------------|--------------|
| Campus 1  | 192.168.0.0/24   | 192.168.0.1  |
| Campus 2  | 192.168.1.0/24   | 192.168.1.1  |
| Campus 3  | 192.168.2.0/24   | 192.168.2.1  |
| Campus 4  | 192.168.3.0/24   | 192.168.3.1  |
| Campus 5  | 192.168.4.0/24   | 192.168.4.1  |

These WAN subnets are used solely for router-to-router communication and external connectivity; VRRP is used to establish redundancy.

### LAN Blocks (Per Campus)

Each campus is allocated a /16 LAN block:

| Campus    | LAN Block       |
|-----------|-----------------|
| Campus 1  | 172.16.0.0/16   |
| Campus 2  | 172.17.0.0/16   |
| Campus 3  | 172.18.0.0/16   |
| Campus 4  | 172.19.0.0/16   |
| Campus 5  | 172.20.0.0/16   |

Within each /16 block, I will subdivide the space into /24 segments for various services.

#### Example - LAN Segmentation for Campus 1

| **Service/Role**                       | **Assigned Subnet** | **Example Device IPs**                                                      |
|----------------------------------------|---------------------|-----------------------------------------------------------------------------|
| **Core Infrastructure** (ESXi, vCenter, NSX Manager, DCs, File Server)  | 172.16.10.0/24      | ESXi Host 1: 172.16.10.11<br>ESXi Host 2: 172.16.10.12<br>ESXi Host 3: 172.16.10.13<br>vCenter: 172.16.10.100<br>NSX Manager: 172.16.10.20–22<br>DCs: 172.16.10.30–31<br>File Server: 172.16.10.40 |
| **Backup Appliances** (Veeam)          | 172.16.15.0/24      | Backup Appliance: 172.16.15.50                                               |
| **Monitoring** (Grafana/Prometheus)      | 172.16.16.0/24      | Monitoring Server: 172.16.16.100                                             |
| **NSX Edge Appliances**                | 172.16.19.0/24      | NSX Edge 1: 172.16.19.1<br>NSX Edge 2: 172.16.19.2                           |
| **(Optional) Guest/IoT**               | 172.16.6.0/24       | Guest/IoT devices                                                           |

A similar approach is used for each campus, adjusting the LAN block according to the campus number (e.g., Campus 2 uses 172.17.0.0/16).

## Campus Configuration Details

Since most campuses are essentially copies of each other (differing only by naming and addressing), the automation uses a shared base configuration alongside campus-specific variable files.

### Example - Campus 1 Variables

```yaml
campus_name: "Campus 1"
wan_subnet: "192.168.0.0/24"
wan_vip: "192.168.0.1"
wan_router_primary: "192.168.0.2"
wan_router_backup: "192.168.0.3"
lan_block: "172.16.0.0/16"
lan_core_subnet: "172.16.10.0/24"
lan_backup_subnet: "172.16.15.0/24"
lan_monitoring_subnet: "172.16.16.0/24"
lan_edge_subnet: "172.16.19.0/24"
```

Campus 2 through Campus 5 will have similar files, replacing the IP subnets accordingly.

## Repository Structure

The repository layout is as follows:

```
multi-campus-network-sim/
├── README.md                      # This document provides a project overview.
├── docs/
│   ├── architecture.md            # Detailed description of the network architecture.
│   ├── addressing-scheme.md       # Detailed WAN and LAN addressing scheme.
│   ├── design-rationale.md        # Explanation of the design decisions.
│   └── diagrams/                  # Network diagrams (images, draw.io files, etc.).
├── configs/
│   ├── base/                      # Base configuration templates common to all campuses.
│   │   ├── wan.cfg                # Base WAN configuration (VRRP, static addressing details).
│   │   └── lan.cfg                # Base LAN segmentation configuration (VLANs, subnets).
│   └── campuses/                  # Campus-specific variable files.
│       ├── campus1.yml
│       ├── campus2.yml
│       ├── campus3.yml
│       ├── campus4.yml
│       └── campus5.yml
├── automation/
│   ├── ansible/                   # Ansible playbooks and Jinja2 templates.
│   │   ├── playbooks/
│   │   │   └── deploy_campuses.yml  # Main playbook for deploying campus configurations.
│   │   └── templates/
│   │       ├── wan_template.j2    # Template for WAN configuration.
│   │       └── lan_template.j2    # Template for LAN segmentation configuration.
│   └── scripts/                   # Utility scripts for deployment or verification.
│       ├── deploy.sh
│       └── verify.sh
├── tests/
│   ├── connectivity_tests.md      # Instructions for testing connectivity.
│   └── failover_tests.md          # Failover and redundancy testing procedures.
└── references/
    └── resources.md               # Links and references for design documentation.
```

## Automation and Templates

- **Automation with Ansible:**  
  The `automation/ansible/` folder contains playbooks and templates that utilize campus-specific variable files to generate final configurations. This reduces duplication and ensures that changes in the base configuration automatically propagate to all campuses.

- **Templates:**  
  Use Jinja2 templates (such as `wan_template.j2` and `lan_template.j2`) to dynamically construct configuration files based on the variables defined in each campus file.

## Deployment and Testing

1. **Deploy WAN Configurations:**  
   Use Ansible playbooks to configure WAN interfaces (with VRRP) for each campus router pair.
2. **Deploy LAN Segmentation:**  
   Push the LAN segmentation (VLAN and subinterface configurations) to the routers.
3. **Testing:**  
   Execute connectivity and failover tests (described in the `tests/` folder) to verify that VRRP operates correctly and that all internal segments are reachable.
4. **Monitoring:**  
   Integrate with tools like Grafana/Prometheus for ongoing health and performance monitoring.

## Future Enhancements

- **SD-WAN Integration:**  
  Evaluate overlay SD-WAN solutions to optimize dynamic WAN routing.
- **IPv6 Deployment:**  
  Although the current focus is IPv4, the plan can later be extended to dual-stack or IPv6.
- **Continuous Automation:**  
  Expand CI/CD pipelines (using GitHub Actions, for example) to include automated tests and configuration validation.

## References

- [Cisco VRRP Documentation](https://www.cisco.com/c/en/us/support/docs/ip/virtual-router-redundancy-protocol-vrrp/13788-vrrp.html)
- [OPNsense Documentation](https://docs.opnsense.org/)
- [Ansible Networking Documentation](https://docs.ansible.com/ansible/latest/network/getting_started/index.html)

---

Happy Networking!
