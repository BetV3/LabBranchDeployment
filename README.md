# Multi-Campus Network Simulation Project

This project simulates a multi-campus network infrastructure in a homelab environment. The design models realistic enterprise scenarios by separating WAN and LAN addressing for each campus and subdividing internal LANs to support various services.

## Project Overview

The objective is to create five independent campus environments, each with its own WAN connectivity and an internally segmented LAN. The WAN interfaces enable external connectivity and redundancy, while the LAN provides a large private address block for internal services such as virtualization, management, user workstations, and specialized application environments.

## Network Architecture

The overall architecture consists of two main parts for each campus:

1. **WAN Connectivity:**  
   - Each campus is assigned its own dedicated WAN subnet (a /24 block).  
   - This subnet is used to connect the campus routers to the upstream network (simulated by an OPNsense router or equivalent).  
   - Redundancy is achieved through VRRP (or a similar protocol) with a shared virtual IP address serving as the campus WAN gateway.

2. **Internal LAN Segmentation:**  
   - Each campus receives a large private LAN block in the /16 range.  
   - This /16 block is subdivided into multiple /24 subnets for different purposes (e.g., infrastructure, servers, workstations, backup, monitoring, etc.).  
   - Internal segmentation improves security, management, and scalability, as well as providing clear boundaries for different functions.

## Addressing Scheme

### WAN Subnets (Per Campus)

Each campus gets a dedicated /24 for its WAN:
- **Campus 1 WAN:** 192.168.0.0/24  
  - **Example WAN Gateway (VIP):** 192.168.0.1  
  - Routers might use addresses such as 192.168.0.2 and 192.168.0.3.
- **Campus 2 WAN:** 192.168.1.0/24  
  - **Example WAN Gateway:** 192.168.1.1  
- **Campus 3 WAN:** 192.168.2.0/24  
  - **Example WAN Gateway:** 192.168.2.1  
- **Campus 4 WAN:** 192.168.3.0/24  
  - **Example WAN Gateway:** 192.168.3.1  
- **Campus 5 WAN:** 192.168.4.0/24  
  - **Example WAN Gateway:** 192.168.4.1  

### LAN Subnets (Per Campus)

Each campus is assigned a large /16 LAN block. This address space will be subdivided into smaller /24 segments tailored for specific functions.

- **Campus 1 LAN:** 172.16.0.0/16  
- **Campus 2 LAN:** 172.17.0.0/16  
- **Campus 3 LAN:** 172.18.0.0/16  
- **Campus 4 LAN:** 172.19.0.0/16  
- **Campus 5 LAN:** 172.20.0.0/16  

#### Example: Internal Subnet Design for Campus 1

Within the 172.16.0.0/16 block for Campus 1, you could allocate:

| **Service/Role**                   | **Assigned Subnet** | **Example Gateway/Key IPs**                                       |
|------------------------------------|---------------------|-------------------------------------------------------------------|
| **Core Infrastructure**            | 172.16.10.0/24      | ESXi Hosts: 172.16.10.11–13<br>vCenter: 172.16.10.100<br>NSX Managers: 172.16.10.20–22<br>Domain Controllers: 172.16.10.30–31<br>File Server: 172.16.10.40 |
| **Backup Appliances**              | 172.16.15.0/24      | Backup Appliance (Veeam): 172.16.15.50                             |
| **Monitoring**                     | 172.16.16.0/24      | Monitoring (Grafana/Prometheus): 172.16.16.100                      |
| **Edge/NSX Appliances**            | 172.16.19.0/24      | NSX Edge Appliances: 172.16.19.1, 172.16.19.2                       |
| **Additional Segments (Optional)** | 172.16.6.0/24, etc. | For Guest, IoT, or other specialized networks                       |

For other campuses, the pattern is similar:
- **Campus 2 LAN:** Use, for example, 172.17.10.0/24 for core infrastructure, 172.17.15.0/24 for backup, etc.
- **Campus 3 LAN:** Use, for example, 172.18.10.0/24 for core infrastructure.
- **Campus 4 LAN:** Use, for example, 172.19.10.0/24 for core infrastructure.
- **Campus 5 LAN:** Use, for example, 172.20.10.0/24 for core infrastructure.

## Campus 1 Configuration Overview (Informational)

### WAN Connectivity (Campus 1)

- **WAN Subnet:** 192.168.0.0/24  
  - **VIP (Gateway):** 192.168.0.1  
  - **Router Addresses:**  
    - Router 1: 192.168.0.2  
    - Router 2: 192.168.0.3  
- **Redundancy:**  
  - VRRP (or equivalent protocol) is used to ensure seamless failover on the WAN interface.

### LAN Segmentation (Campus 1)

- **LAN Block:** 172.16.0.0/16  
  - This block is subdivided into smaller subnets for different services:
    - **Core Infrastructure (ESXi, vCenter, NSX Managers, Domain Controllers, File Server):** 172.16.10.0/24  
      - **Example Assignments:**  
        - ESXi Host 1: 172.16.10.11  
        - ESXi Host 2: 172.16.10.12  
        - ESXi Host 3: 172.16.10.13  
        - vCenter Server Appliance: 172.16.10.100  
        - NSX Manager Cluster Nodes: 172.16.10.20–22  
        - Domain Controllers: 172.16.10.30–31  
        - Windows File Server: 172.16.10.40  
    - **Backup Appliances (Veeam):** 172.16.15.0/24  
      - **Example:** Veeam Backup: 172.16.15.50  
    - **Monitoring (Grafana/Prometheus):** 172.16.16.0/24  
      - **Example:** Monitoring Server: 172.16.16.100  
    - **NSX Edge Appliances:** 172.16.19.0/24  
      - **Example:** NSX Edge Appliance 1: 172.16.19.1  
                   NSX Edge Appliance 2: 172.16.19.2  

### Other Campuses

The same design pattern applies to campuses 2 through 5 using their respective WAN and LAN blocks:

- **Campus 2:**  
  - **WAN:** 192.168.1.0/24 (VIP: 192.168.1.1)  
  - **LAN:** 172.17.0.0/16, with internal segmentation (e.g., 172.17.10.0/24 for core infrastructure)
- **Campus 3:**  
  - **WAN:** 192.168.2.0/24 (VIP: 192.168.2.1)  
  - **LAN:** 172.18.0.0/16
- **Campus 4:**  
  - **WAN:** 192.168.3.0/24 (VIP: 192.168.3.1)  
  - **LAN:** 172.19.0.0/16
- **Campus 5:**  
  - **WAN:** 192.168.4.0/24 (VIP: 192.168.4.1)  
  - **LAN:** 172.20.0.0/16

## Configuration Automation

To streamline deployment, all configurations can be automated using tools such as Ansible. Templates are created for:

- **WAN Interface Configuration:** Assigning static IPs and VRRP settings.
- **LAN Segmentation:** Defining VLAN interfaces (or subinterfaces) for the various service subnets.
- **Variables:** Each campus has its own variables for both the WAN and LAN addressing schemes.

**Example Template Variables (for Campus 1):**

```yaml
campus: "Campus 1"
wan_subnet: "192.168.0.0/24"
wan_vip: "192.168.0.1"
wan_router_primary: "192.168.0.2"
wan_router_backup: "192.168.0.3"
lan_block: "172.16.0.0/16"
lan_core_subnet: "172.16.10.0/24"  # For ESXi, vCenter, NSX, DCs, File Server
lan_backup_subnet: "172.16.15.0/24"
lan_monitoring_subnet: "172.16.16.0/24"
lan_edge_subnet: "172.16.19.0/24"
```

The playbooks then deploy configurations for each campus router pair according to these variables.

## Deployment and Testing

1. **Deploy WAN Configurations:**  
   Using your automation scripts, configure the WAN interfaces on all campus routers with VRRP for redundancy.
2. **Deploy LAN Segmentation:**  
   Push VLAN or subinterface configurations to each router for internal LAN segmentation according to the design.
3. **Test Failover and Connectivity:**  
   - Verify that if the primary router goes down, the VRRP-enabled VIP takes over properly.  
   - Check connectivity between LAN subnets and external connections.
4. **Monitor:**  
   Use integrated tools (such as Grafana/Prometheus) to monitor network performance and ensure correct routing and segmentation.

## Future Enhancements

- **Integration with SD-WAN:**  
  Evaluate overlay solutions to further simplify WAN management.
- **IPv6 Implementation:**  
  Although the current focus is IPv4, consider extending to dual-stack in the future.
- **Continuous Automation:**  
  Expand automation scripts to include post-deployment verification and configuration updates.

## References

- [Network Addressing Fundamentals](https://www.cisco.com/c/en/us/support/docs/ip/routing-information-protocol-rip/13714-22.html)
- [Ansible for Network Automation](https://docs.ansible.com/ansible/latest/network/getting_started/index.html)
- [VRRP Overview and Best Practices](https://www.cisco.com/c/en/us/support/docs/ip/virtual-router-redundancy-protocol-vrrp/13788-vrrp.html)

---
