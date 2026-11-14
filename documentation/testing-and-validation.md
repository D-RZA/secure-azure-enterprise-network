# Network Testing and Validation

## 1. Overview

Testing was carried out to verify the network configuration, security controls, routing behaviour and private DNS resolution.

The validation evidence comes from two stages of the project:

- Original enterprise deployment in the previous Azure subscription.
- Selected network and DNS validation in the current Azure subscription.

## 2. Original Deployment Tests

### 2.1 VNet Peering

Hub-to-Spoke peering was configured between the Hub and five departmental VNets.

The design did not include direct Spoke-to-Spoke peering.

### 2.2 Effective Route Validation

Effective routes were inspected on the IT and Production VM network interfaces.

| Source VM | Destination Prefix | Next Hop |
|---|---|---|
| `vm-it-admin-01` | `10.0.128.0/19` | Virtual appliance `10.0.0.4` |
| `vm-production-linux-01` | `10.0.32.0/19` | Virtual appliance `10.0.0.4` |

These results confirmed that the selected routes directed traffic through Azure Firewall.

### 2.3 SSH Connectivity

An SSH connection was successfully established from the IT administration VM to the Production Linux VM.

The connection used SSH key authentication and the Production VM's private IP address.

The successful login confirmed the intended IT-to-Production administrative access path in the original deployment.

## 3. Current Subscription Tests

### 3.1 Private DNS Resolution

The current environment contains the IT and Production VNets, their Private DNS links and a dedicated Linux test VM.

The following command was executed:

```bash
getent hosts vm-production-linux-01.secure-enterprise.internal
```

Successful output:

```text
10.0.128.4 vm-production-linux-01.secure-enterprise.internal
```

The test VM's own hostname was also checked:

```bash
getent hosts vm-dns-test-01.secure-enterprise.internal
```

Successful output:

```text
10.0.32.4 vm-dns-test-01.secure-enterprise.internal
```

Both lookups confirmed successful private hostname resolution from the IT VNet.

### 3.2 SSH Service Status

The SSH service was inspected using:

```bash
sudo systemctl status ssh
```

The service was confirmed active and listening on TCP port 22.

### 3.3 Network Watcher Connection Troubleshoot

A diagnostic was executed against the test VM's own private IP address.

| Diagnostic | Result |
|---|---|
| Source | `vm-dns-test-01` |
| Destination | `10.0.32.4` |
| Protocol | TCP |
| Destination Port | 22 |
| Connectivity Test | Unreachable |
| Outbound NSG Diagnostic | Allow |
| Next Hop | Success — Virtual Network |

The connectivity test did not establish a TCP connection. The outbound NSG diagnostic and next-hop result were successful.

This result is documented as an unsuccessful connectivity test, not as proof of successful SSH access.

## 4. Validation Summary

| Test | Result |
|---|---|
| Original Hub-to-Spoke peering | Configured |
| Original IT-to-Production SSH | Successful |
| Original effective route inspection | Verified |
| Current Private DNS resolution | Successful |
| Current SSH service status | Active |
| Current Network Watcher TCP test | Unsuccessful |
| Current interactive SSH login | Not yet validated |

## 5. Outstanding Work

The following tasks remain:

- Complete an interactive SSH login test using the DNS hostname.
- Investigate the unsuccessful Network Watcher connectivity test.
- Revalidate the complete enterprise architecture in one active subscription.
- Expand network monitoring and automated infrastructure deployment.

## 6. Conclusion

The project provided practical experience with Azure network design, traffic control, routing, private DNS and troubleshooting.

The original deployment demonstrated the intended administrative SSH path between the IT and Production networks. The current environment provided additional validation of Private DNS resolution.

The test results are recorded separately to maintain a clear distinction between successful configurations, verified connectivity and outstanding work.
