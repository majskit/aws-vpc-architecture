# Multi-Tier VPC Architecture Diagram

```mermaid
graph TB
    Internet((("🌐 Internet")))
    
    subgraph AWS["AWS Cloud"]
        subgraph VPC["Lab VPC (10.0.0.0/16)"]
            IGW["Internet Gateway<br/>Lab IGW"]
            
            subgraph PublicSubnet["Public Subnet (10.0.0.0/24)"]
                Bastion["🖥️ Bastion Server<br/>t3.micro<br/>Public IP: Auto-assigned"]
                NAT["NAT Gateway<br/>Elastic IP"]
            end
            
            subgraph PrivateSubnet["Private Subnet (10.0.2.0/23)"]
                Private["🖥️ Private Instance<br/>t3.micro<br/>No Public IP"]
            end
            
            subgraph RouteTables["Route Tables"]
                PubRT["Public Route Table<br/>10.0.0.0/16 → local<br/>0.0.0.0/0 → IGW"]
                PrivRT["Private Route Table<br/>10.0.0.0/16 → local<br/>0.0.0.0/0 → NAT GW"]
            end
            
            subgraph SecurityGroups["Security Groups"]
                BastionSG["Bastion SG<br/>SSH 22 from 0.0.0.0/0"]
                PrivateSG["Private Instance SG<br/>SSH 22 from 10.0.0.0/16"]
            end
        end
    end
    
    Internet <-->|"Inbound & Outbound"| IGW
    IGW <--> Bastion
    IGW <--> NAT
    NAT -->|"Outbound only"| Private
    Bastion -->|"SSH port 22"| Private
    
    PubRT -.->|"Associated"| PublicSubnet
    PrivRT -.->|"Associated"| PrivateSubnet
    BastionSG -.->|"Applied to"| Bastion
    PrivateSG -.->|"Applied to"| Private
    
    style VPC fill:#e8f4fd,stroke:#1a73e8,stroke-width:2px
    style PublicSubnet fill:#e6f4ea,stroke:#34a853,stroke-width:2px
    style PrivateSubnet fill:#fce8e6,stroke:#ea4335,stroke-width:2px
    style IGW fill:#fff3e0,stroke:#fb8c00,stroke-width:2px
    style NAT fill:#fff3e0,stroke:#fb8c00,stroke-width:2px
    style Bastion fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style Private fill:#ffebee,stroke:#c62828,stroke-width:2px
```

## Traffic Flow

1. **User → Bastion**: User connects via SSH through Internet Gateway to Bastion Server in Public Subnet
2. **Bastion → Private Instance**: SSH from Bastion to Private Instance using private IP
3. **Private Instance → Internet**: Outbound traffic goes through NAT Gateway (for updates, patches) — no inbound from internet
