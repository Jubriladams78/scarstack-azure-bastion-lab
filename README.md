
# Secure Azure Ubuntu VM Deployment — Nginx + Azure Bastion

Enterprise-grade deployment of a hardened Ubuntu Linux web server on Microsoft Azure. All administrative access flows through **Azure Bastion**, while the Nginx web application is served publicly over HTTP via an Azure DNS name.

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
                     └── vnet-scarstack-security-prod (10.1.x.x)
```

**Key security decisions:**

- **Intended design: SSH only through Bastion.** The VM was created with no public inbound ports, so administration runs over Bastion's private connection (see Post-lab review below)
- **Subnet-level NSG** — traffic rules enforced at the subnet (`nsg-scarstack-production`), not the NIC, so policy scales to every future VM in the subnet
- **Governance tags** — Department, Environment, Project, Owner, and CostCenter applied for cost tracking and ownership

## Deployed Resources

| Resource | Name | Notes |
|---|---|---|
| Resource Group | `rg-scarstack-security-prod` | Container for all lab resources |
| Virtual Network | `vnet-scarstack-security-prod` | 10.1.x.x address space (exact prefix not recorded) |
| Subnet | `subnet-production` | `10.1.0.0/26` |
| Azure Bastion | enabled on VNet | Secure browser-based SSH |
| NSG | `nsg-scarstack-production` | Allows 22/80/443, associated to subnet. Port 22 should have been limited to the Bastion subnet (see Post-lab review) |
| Virtual Machine | `vm-linux-nginx-prod` | Ubuntu LTS, Standard B2s, Zone 1, Premium SSD |
| Public IP + DNS | `scarstack-nginx-prod.<region>.cloudapp.azure.com` | Public web access |

## Deployment Walkthrough

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

- ✅ Bastion SSH session to `vm-linux-nginx-prod` (over the VM's private IP)
- ✅ Nginx `active (running)`
- ✅ Welcome page reachable via public IP
- ✅ Welcome page reachable via `http://scarstack-nginx-prod.<region>.cloudapp.azure.com`

## Post-lab Review

On review, I found that the subnet NSG allowed inbound SSH (port 22) without restricting the source to the Bastion subnet, while the VM had a public IP. That means SSH was likely reachable from the internet even though Bastion was in place. The correct configuration limits the port 22 rule's source to the AzureBastionSubnet range, or removes the rule entirely, since Bastion reaches the VM over the virtual network. The environment has since been decommissioned.

## Skills Demonstrated

Azure Resource Groups · Virtual Networks & Subnetting · Network Security Groups · Azure Bastion · Zero Trust access patterns · Ubuntu Server Administration · Nginx · Public IP & DNS configuration · Governance Tagging

## About Me

IT support professional in transition. Certified in **AZ-104**, **AWS Solutions Architect Associate**, **Security+**, and **CompTIA Cloud+**. Completing a B.S. in Cloud and Network Engineering at Western Governors University (WGU), expected December 2027.

📧 adam.jubril78@gmail.com
