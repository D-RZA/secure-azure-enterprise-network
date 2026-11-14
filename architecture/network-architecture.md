
# Azure Enterprise Network Architecture

This diagram represents the original Hub-and-Spoke network design implemented for this project.

```mermaid
flowchart TB
    Internet((Internet))

    subgraph HUB["Hub VNet — 10.0.0.0/19"]
        FW["Azure Firewall<br/>10.0.0.4"]
        BAS["Azure Bastion"]
        MGMT["Management Subnet"]
        SHARED["Shared Services Subnet"]
    end

    subgraph IT["IT Spoke — 10.0.32.0/19"]
        ITVM["IT Admin VM<br/>10.0.32.4"]
    end

    subgraph FIN["Finance Spoke — 10.0.64.0/19"]
        FINAPP["Application Subnet"]
        FINDATA["Database Subnet"]
    end

    subgraph SALES["Sales Spoke — 10.0.96.0/19"]
        SALESAPP["Application Subnet"]
        SALESMGMT["Management Subnet"]
    end

    subgraph PROD["Production Spoke — 10.0.128.0/19"]
        PRODVM["Production Linux VM<br/>10.0.128.4"]
        PRODAPP["Application Subnet"]
        PRODDATA["Database Subnet"]
    end

    subgraph DEV["Development Spoke — 10.0.160.0/19"]
        DEVAPP["Development Subnet"]
        DEVTEST["Testing Subnet"]
    end

    IT <-->|VNet Peering| HUB
    FIN <-->|VNet Peering| HUB
    SALES <-->|VNet Peering| HUB
    PROD <-->|VNet Peering| HUB
    DEV <-->|VNet Peering| HUB

    ITVM --> FW
    FW --> PRODVM
    Internet --> FW
    BAS --> ITVM
    BAS --> PRODVM
```

## Network Design Notes

- The Hub provides centralized network security and shared services.
- Five departmental Spoke VNets connect to the Hub through VNet peering.
- Direct Spoke-to-Spoke peering is not part of the design.
- Azure Firewall provides centralized traffic inspection.
- User Defined Routes direct workload traffic through the firewall.
- Azure Bastion provides administrative access to private virtual machines.
