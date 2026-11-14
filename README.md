# Secure Azure Enterprise Network Architecture

## About the Project

I built this project to get practical experience designing and securing enterprise network infrastructure in Microsoft Azure.

The goal was to create a network that could support different departments while keeping their workloads separated, controlling how they communicate, and providing a secure way to manage resources.

I used a Hub-and-Spoke architecture, with a central network handling shared services and security, while separate spoke networks support IT, Finance, Sales, Production and Development.

This project covers the design, configuration, testing and troubleshooting of the network.

## Architecture Overview

The original network design uses six Virtual Networks:

| Virtual Network | Address Space | Purpose |
|---|---|---|
| Hub | `10.0.0.0/19` | Central security and shared services |
| IT | `10.0.32.0/19` | IT administration |
| Finance | `10.0.64.0/19` | Finance workloads |
| Sales | `10.0.96.0/19` | Sales workloads |
| Production | `10.0.128.0/19` | Production workloads |
| Development | `10.0.160.0/19` | Development and testing |

The design contains 14 subnets distributed across the six VNets.

The Hub acts as the central point for network security and shared services. The spoke networks are connected to the Hub through VNet peering rather than being directly connected to one another.

## Azure Services and Configuration

### 1. Virtual Networks and Subnetting

Created separate virtual networks for each department and divided them into smaller subnets based on workload requirements.

The addressing plan was designed to avoid overlapping IP ranges and allow room for future expansion.

### 2. VNet Peering

Configured Hub-to-Spoke peering to enable communication between the central network and departmental networks.

The design avoids direct Spoke-to-Spoke peering, allowing the Hub to serve as the central network path.

### 3. Azure Firewall

Deployed Azure Firewall in the Hub network to provide centralized traffic inspection and network-level access control.

A firewall policy was configured with a rule allowing SSH administration from the IT subnet to a specific Production workload.

Example rule:

| Setting | Value |
|---|---|
| Rule name | `allow-ssh-from-it` |
| Source | `10.0.32.0/25` |
| Destination | `10.0.128.4` |
| Protocol | TCP |
| Port | 22 |
| Action | Allow |

### 4. Network Security Groups

Configured NSGs to control traffic at the subnet and network interface levels.

For example, SSH access to the Production Linux VM was restricted to the IT management subnet rather than being open to all sources.

### 5. User Defined Routes

Created route tables for the spoke networks and configured routes to direct traffic through the Azure Firewall private IP address:

`10.0.0.4`

The routing configuration included default routes and routes for communication with other spoke networks.

### 6. Azure Bastion

Deployed Azure Bastion in the Hub network to provide browser-based access to private virtual machines.

The VMs were configured without public IP addresses, reducing their exposure to direct internet access.

### 7. Azure Private DNS

Configured a Private DNS zone named:

`secure-enterprise.internal`

Created A records for the IT and Production hostnames and linked the DNS zone to the IT and Production virtual networks.

Automatic registration was disabled because the records were managed manually.

## Testing and Validation

I used a combination of Azure tools and Linux commands to check the network configuration.

### Private DNS resolution

From the IT test VM, I ran:

```bash
getent hosts vm-production-linux-01.secure-enterprise.internal
```

The command successfully returned:

```text
10.0.128.4 vm-production-linux-01.secure-enterprise.internal
```

I also tested the hostname of the test VM itself:

```bash
getent hosts vm-dns-test-01.secure-enterprise.internal
```

Result:

```text
10.0.32.4 vm-dns-test-01.secure-enterprise.internal
```

Both lookups were successful, confirming that the Private DNS zone and its IT VNet link were working.

### SSH service check

Checked the SSH service on the Linux test VM using:

```bash
sudo systemctl status ssh
```

The service was active and listening on TCP port 22.

### Azure Network Watcher

Used Connection Troubleshoot to investigate TCP connectivity to the test VM's own private IP address on port 22.

| Diagnostic | Result |
|---|---|
| Connectivity test | Unreachable |
| Outbound NSG diagnostic | Allow |
| Next hop | Success — Virtual Network |

The TCP connectivity test was unsuccessful, despite the outbound NSG and routing checks passing. This result is retained as part of the troubleshooting evidence rather than being presented as a successful connectivity test.

## Deployment Status and Environment Notes

The original six-VNet enterprise architecture, including the firewall, Bastion, routing and VM security configurations, was deployed in an earlier Azure subscription.

After that subscription became unavailable, I created a new Azure subscription to continue the project without rebuilding the entire environment.

In the new subscription, I recreated the IT and Production VNets, configured the Private DNS zone and VNet links, deployed a dedicated Linux DNS test VM, and successfully validated private hostname resolution.

The original Production and IT VMs have not been recreated in the new subscription. Their DNS records preserve the original architecture's addressing plan; they should not be interpreted as proof that those original VMs are currently running.

## Technologies Used

- Microsoft Azure
- Azure Virtual Networks
- VNet Peering
- Azure Firewall and Firewall Policy
- Network Security Groups
- User Defined Routes
- Azure Bastion
- Azure Private DNS
- Azure Network Watcher
- Ubuntu Linux
- SSH

## What I Learned

Working through this project gave me practical experience with:

- Designing Hub-and-Spoke network architectures.
- Planning IPv4 address spaces and subnet allocations.
- Controlling network traffic with NSGs and Azure Firewall.
- Understanding routing and next-hop behaviour.
- Managing private access to Azure virtual machines.
- Configuring private DNS and testing hostname resolution.
- Using Network Watcher to investigate connectivity problems.
- Documenting implementation decisions and troubleshooting results.

## Future Improvements

Some areas I intend to improve include:

- Revalidating the complete enterprise architecture in one active subscription.
- Completing end-to-end SSH testing using private DNS hostnames.
- Automating infrastructure deployment with Terraform.
- Expanding network monitoring and security logging.

## Project Goal

This project is part of my journey into Cloud Engineering and Cloud Security.

My immediate goal is to develop stronger practical skills in Azure networking and security. In the long term, I want to build the experience required to become a Cloud Architect.
