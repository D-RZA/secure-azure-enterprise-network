# Network Security Controls

## 1. Overview

Security was implemented using multiple layers of network protection. Azure Firewall provides centralized traffic inspection, while Network Security Groups (NSGs) control traffic at the subnet and network-interface levels.

Administrative access to the production VM was restricted to the IT management subnet.

## 2. Azure Firewall

| Setting | Configuration |
|---|---|
| Firewall | `fw-hub-eastus` |
| SKU | Standard |
| Private IP | `10.0.0.4` |
| Firewall Policy | `fw-policy-hub-eastus` |
| Policy SKU | Standard |

### Firewall Rule

The firewall policy contains a network rule allowing SSH administration from the IT subnet to the production Linux VM.

| Setting | Value |
|---|---|
| Rule Collection Group | `rcg-enterprise-security` |
| Group Priority | `200` |
| Rule Collection | `nrc-linux-administration` |
| Collection Priority | `200` |
| Rule Name | `allow-ssh-from-it` |
| Rule Type | Network |
| Action | Allow |
| Protocol | TCP |
| Source | `10.0.32.0/25` |
| Destination | `10.0.128.4` |
| Destination Port | `22` |

This limits the configured SSH firewall permission to the IT management subnet and the production VM.

## 3. Network Security Groups

### Production VM NSG

Resource: `vm-production-linux-01-nsg`

| Setting | Value |
|---|---|
| Rule Name | `default-allow-ssh` |
| Priority | `1000` |
| Direction | Inbound |
| Protocol | TCP |
| Source | `10.0.32.0/25` |
| Destination Port | `22` |
| Action | Allow |

This permits SSH access from the IT management subnet rather than allowing SSH from any source.

### IT Administration VM NSG

Resource: `vm-it-admin-01-nsg`

The custom inbound SSH rule that initially allowed access from Any was removed. The NSG was left with its default security rules.

## 4. Virtual Machine Access

Both VMs in the original deployment were configured without public IP addresses.

- `vm-it-admin-01` — `10.0.32.4`
- `vm-production-linux-01` — `10.0.128.4`

Administrative access was intended to originate from the IT management environment.

Azure Bastion provided browser-based access to the private VMs.

## 5. Security Validation

The following administrative access test was completed in the original deployment:

1. Generated an SSH key pair from the IT VM.
2. Added the public key to the production VM.
3. Corrected the private key ownership and permissions.
4. Connected from the IT VM to the production VM using SSH.
5. Confirmed successful login to `vm-production-linux-01` as `azureuser`.

The successful connection validated the configured IT-to-Production SSH access path.

## 6. Security Design Principles

- No public IP addresses on the workload VMs.
- Centralized traffic inspection through Azure Firewall.
- Restricted administrative access using source subnet controls.
- Layered security using Firewall rules and NSGs.
- Private VM administration through Azure Bastion.

## 7. Deployment Scope

These configurations describe the original six-VNet deployment in the previous Azure subscription. The smaller environment in the current subscription is being used for selected additional tests and should not be mistaken for a recreation of the original security infrastructure.
