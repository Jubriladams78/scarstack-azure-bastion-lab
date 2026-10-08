
# Secure Azure Ubuntu VM Deployment — Nginx + Azure Bastion

Enterprise-grade deployment of a hardened Ubuntu Linux web server on Microsoft Azure, with **zero public SSH exposure**. All administrative access flows through **Azure Bastion**, while the Nginx web application is served publicly over HTTP via an Azure DNS name.

> 🎓 Built as part of the **Scarstack Solutions IT Bootcamp** (Azure Cloud Solution Architect track, July 2026 Cohort).

---

## Business Scenario

Jularex Panels Inc. is migrating internal web applications to Azure for scalability, security, and remote accessibility. Compliance policy prohibits exposing SSH to the public internet — administration must occur exclusively through Azure Bastion, while the web tier remains publicly reachable via a DNS name.

## Architecture

```
Internet ──HTTP:80──▶ Public IP + DNS label ──▶ Subnet NSG ──▶ Ubuntu VM (nginx)
                                                                │ subnet-production 10.1.0.0/26
Admin ──HTTPS:443──▶ Azure Bastion ──SSH (private IP)──▶───────┘
                     │ AzureBastionSubnet
                     └── vnet-scarstack-security-prod 10.1.0.0/14
```

**Key security decisions:**

- **No public SSH** — the VM is created with zero public inbound ports; SSH happens over Bastion's private connection
- **Subnet-level NSG** — traffic rules enforced at the subnet (`nsg-scarstack-production`), not the NIC, so policy scales to every future VM in the subnet
- **Governance tags** — Department, Environment, Project, Owner, and CostCenter applied for cost tracking and ownership

## Deployed Resources

| Resource | Name | Notes |
|---|---|---|
| Resource Group | `rg-scarstack-security-prod` | Container for all lab resources |
| Virtual Network | `vnet-scarstack-security-prod` | `10.1.0.0/14` |
| Subnet | `subnet-production` | `10.1.0.0/26` |
| Azure Bastion | enabled on VNet | Secure browser-based SSH |
| NSG | `nsg-scarstack-production` | Allows 22/80/443, associated to subnet |
| Virtual Machine | `vm-linux-nginx-prod` | Ubuntu LTS, Standard B2s, Zone 1, Premium SSD |
| Public IP + DNS | `scarstack-nginx-prod.<region>.cloudapp.azure.com` | Public web access |

## Deployment Walkthrough

Full step-by-step instructions, configuration values, and required screenshots:

📄 **[lab-01-secure-ubuntu-nginx-bastion.md](lab-01-secure-ubuntu-nginx-bastion.md)**

Summary of phases:

1. Create the Resource Group
2. Create the VNet with `subnet-production`, Azure Bastion, and governance tags
3. Create the NSG (SSH/HTTP/HTTPS) and associate it to the subnet
4. Deploy the Ubuntu VM — public inbound ports **None**, NIC NSG **None**
5. Connect via Azure Bastion
6. Install and enable Nginx
7. Configure the public IP DNS name label
8. Verify the Nginx welcome page via public IP and DNS name

## Nginx Installation

```bash
sudo apt update -y
sudo apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl status nginx
```

## Verification

- ✅ Bastion SSH session to `vm-linux-nginx-prod` (no port 22 exposed publicly on the VM)
- ✅ Nginx `active (running)`
- ✅ Welcome page reachable via public IP
- ✅ Welcome page reachable via `http://scarstack-nginx-prod.<region>.cloudapp.azure.com`

Screenshots for every phase are in [`screenshots/`](screenshots/).

## Skills Demonstrated

Azure Resource Groups · Virtual Networks & Subnetting · Network Security Groups · Azure Bastion · Zero Trust access patterns · Ubuntu Server Administration · Nginx · Public IP & DNS configuration · Governance Tagging

## About Me

IT support professional in transition. Certified in **AZ-104**, **AWS Solutions Architect Associate**, **Security+**, and **CompTIA Cloud+**. Completing a B.S. in Cloud and Network Engineering at Western Governors University (WGU), expected December 2027.

📧 adam.jubril78@gmail.com
