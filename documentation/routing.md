# Azure Network Routing Configuration

## 1. Overview

The original network used User Defined Routes (UDRs) to direct traffic from the departmental spoke networks through the central Azure Firewall in the hub.

This design provides centralized traffic inspection and avoids direct routing between departmental spokes.

## 2. Central Firewall

| Setting | Value |
|---|---|
| Firewall | `fw-hub-eastus` |
| Hub VNet | `vnet-hub-eastus` |
| Firewall Private IP | `10.0.0.4` |
| Routing Method | User Defined Routes |
| Next Hop Type | Virtual Appliance |

## 3. Route Tables

Five route tables were created:

| Spoke | Route Table |
|---|---|
| IT | `rt-spoke-to-firewall` |
| Finance | `rt-finance-to-firewall` |
| Sales | `rt-sales-to-firewall` |
| Production | `rt-production-to-firewall` |
| Development | `rt-development-to-firewall` |

Each route table contained five routes: one default route and four routes for the other departmental spoke address ranges.

## 4. Routing Configuration

The following route configuration was used:

| Destination Prefix | Next Hop Type | Next Hop Address |
|---|---|---|
| `0.0.0.0/0` | Virtual Appliance | `10.0.0.4` |
| `10.0.32.0/19` | Virtual Appliance | `10.0.0.4` |
| `10.0.64.0/19` | Virtual Appliance | `10.0.0.4` |
| `10.0.96.0/19` | Virtual Appliance | `10.0.0.4` |
| `10.0.128.0/19` | Virtual Appliance | `10.0.0.4` |
| `10.0.160.0/19` | Virtual Appliance | `10.0.0.4` |

**Note:** Each route table contains the default route and the four remote Spoke routes. Its own VNet prefix is not included as a route through the firewall.

## 5. Subnet Associations

The route tables were associated with the workload subnets across all five spoke VNets.

This ensures that traffic leaving those subnets follows the intended routing configuration.

## 6. Route Validation

Effective routes were inspected on the network interfaces of the IT and Production VMs.

The following routes were confirmed:

- IT VM: `10.0.128.0/19` routed through virtual appliance `10.0.0.4`.
- Production VM: `10.0.32.0/19` routed through virtual appliance `10.0.0.4`.

The effective route checks confirmed that traffic between the IT and Production address ranges was directed toward the central firewall.

## 7. Connectivity Validation

An SSH connection was successfully established from the IT administration VM to the Production Linux VM.

This demonstrated that the configured routing and security controls supported the intended administrative connection.

## 8. Design Principles

- Centralized routing through the Hub VNet.
- Azure Firewall as the next hop for spoke traffic.
- No direct Spoke-to-Spoke peering.
- Consistent routing policy across departmental networks.
- Effective route inspection to validate forwarding decisions.

## 9. Deployment Scope

This document describes the original six-VNet deployment in the previous Azure subscription. The current subscription contains a smaller test environment and does not reproduce all five route tables.
