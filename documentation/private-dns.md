# Azure Private DNS Configuration

## 1. Overview

Azure Private DNS was configured to provide internal name resolution for resources within the private network.

The DNS zone uses a custom internal namespace rather than a publicly accessible domain.

## 2. Private DNS Zone

| Setting | Value |
|---|---|
| DNS Zone | `secure-enterprise.internal` |
| Resource Type | Azure Private DNS Zone |
| Resource Group | `rg-secure-enterprise-network` |
| Region | East US |

## 3. DNS Records

The following A records were created in the private DNS zone:

| Hostname | Record Type | IP Address | TTL |
|---|---|---|---|
| `vm-it-admin-01` | A | `10.0.32.4` | 3600 seconds |
| `vm-production-linux-01` | A | `10.0.128.4` | 3600 seconds |

These records preserve the original deployment's VM naming and addressing references.

## 4. Virtual Network Links

Two virtual network links were created:

| Link Name | Virtual Network | Status |
|---|---|---|
| `link-it-spoke-eastus` | `IT-spoke-VNet` | Completed |
| `link-production-spoke-eastus` | `Production-Spoke-VNet` | Completed |

Auto-registration was disabled on both links.

This allows the linked virtual networks to use the private DNS zone for name resolution without automatically registering VM records.

## 5. DNS Resolution Test

A Linux VM named `vm-dns-test-01` was deployed in the IT network.

The following command was executed using Azure Run Command:

```bash
getent hosts vm-dns-test-01.secure-enterprise.internal
```

The command returned:

```text
10.0.32.4 vm-dns-test-01.secure-enterprise.internal
```

This confirmed that the Linux environment successfully resolved the supplied fully qualified domain name to the expected private IP address.

## 6. Additional SSH Testing

SSH service status was inspected using:

```bash
sudo systemctl status ssh
```

The output confirmed that the SSH service was active and listening on TCP port 22.

An attempted SSH connection through Azure Run Command did not establish an interactive session because Run Command does not provide an interactive terminal.

## 7. Deployment Scope

The DNS zone, virtual network links, and DNS test VM belong to the smaller environment created in the current Azure subscription.

The original six-VNet architecture and its VM records are documented separately. The DNS test does not establish that the original production VM is currently deployed or reachable.
