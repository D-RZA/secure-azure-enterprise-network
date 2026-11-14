
# Network Addressing Plan

## 1. Overview

The original Azure enterprise network uses the private address range `10.0.0.0/16`.

The address space is divided into `/19` blocks, with one block assigned to the hub and each of the five departmental spoke networks. This provides room for future subnet expansion without overlapping existing networks.

## 2. Virtual Network Allocation

| Virtual Network | Address Space |
|---|---|
| Hub VNet | `10.0.0.0/19` |
| IT Spoke VNet | `10.0.32.0/19` |
| Finance Spoke VNet | `10.0.64.0/19` |
| Sales Spoke VNet | `10.0.96.0/19` |
| Production Spoke VNet | `10.0.128.0/19` |
| Development Spoke VNet | `10.0.160.0/19` |

The remaining ranges, `10.0.192.0/19` and `10.0.224.0/19`, are reserved for future expansion.

## 3. Subnet Allocation

### Hub VNet — `10.0.0.0/19`

| Subnet | Address Prefix | Purpose |
|---|---|---|
| AzureFirewallSubnet | `10.0.0.0/26` | Azure Firewall deployment |
| AzureBastionSubnet | `10.0.0.64/26` | Azure Bastion deployment |
| ManagementSubnet | `10.0.1.0/24` | Central management |
| SharedServicesSubnet | `10.0.2.0/24` | Shared network services |

### IT Spoke — `10.0.32.0/19`

| Subnet | Address Prefix | Purpose |
|---|---|---|
| ITManagementSubnet | `10.0.32.0/25` | IT administration workloads |

### Finance Spoke — `10.0.64.0/19`

| Subnet | Address Prefix | Purpose |
|---|---|---|
| ApplicationSubnet | `10.0.64.0/26` | Finance application workloads |
| DatabaseSubnet | `10.0.64.64/27` | Finance database workloads |

### Sales Spoke — `10.0.96.0/19`

| Subnet | Address Prefix | Purpose |
|---|---|---|
| ApplicationSubnet | `10.0.96.0/26` | Sales application workloads |
| ManagementSubnet | `10.0.96.64/28` | Sales management workloads |

### Production Spoke — `10.0.128.0/19`

| Subnet | Address Prefix | Purpose |
|---|---|---|
| WebSubnet | `10.0.128.0/26` | Production web workloads |
| ApplicationSubnet | `10.0.128.64/26` | Production application workloads |
| DatabaseSubnet | `10.0.128.128/27` | Production database workloads |

### Development Spoke — `10.0.160.0/19`

| Subnet | Address Prefix | Purpose |
|---|---|---|
| DevelopmentSubnet | `10.0.160.0/26` | Development workloads |
| TestingSubnet | `10.0.160.64/27` | Testing workloads |

## 4. Addressing Considerations

- Each VNet has a separate, non-overlapping address range.
- Subnets are sized according to their intended workloads.
- Azure reserves five IP addresses in every subnet.
- The `/19` allocation leaves room for additional subnets and services.
- The address plan supports centralized routing through the hub network.

## 5. Original VM Private IP Addresses

| Virtual Machine | Private IP | Network |
|---|---|---|
| `vm-it-admin-01` | `10.0.32.4` | IT Spoke |
| `vm-production-linux-01` | `10.0.128.4` | Production Spoke |

These addresses belong to the original six-VNet implementation.

## 6. Implementation Note

The addressing plan documents the original hub-and-spoke deployment. The current subscription contains a smaller test environment used to continue validating selected services, including Private DNS.
