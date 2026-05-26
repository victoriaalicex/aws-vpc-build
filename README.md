# AWS VPC Build — Cloud Infrastructure Project

A custom AWS Virtual Private Cloud (VPC) built from scratch demonstrating core cloud networking fundamentals. This project forms the infrastructure foundation for the F1 Pit Lane Dashboard (Project 2).

---

## What I Built

A fully functional cloud network environment on AWS consisting of:

- Custom VPC with a defined private IP space
- Public subnet carved from the VPC CIDR block
- EC2 virtual machine instance running Amazon Linux 2023
- Internet Gateway enabling outbound internet connectivity
- Route table configured to direct traffic through the IGW
- Security group restricting SSH access to my IP only

---

## Architecture

```
Internet
    │
Internet Gateway (my-igw)
    │
VPC: 10.0.0.0/16
    │
Public Subnet: 10.0.1.0/24
    │
EC2 Instance (t3.micro) ← Security Group (port 22, my IP only)
```

---

## Screenshots

### Custom VPC
![VPC](screenshots/vpc.png)

### Public Subnet
![Subnet](screenshots/subnet.png)

### Internet Gateway Attached
![IGW](screenshots/igw.png)

### Route Table
![Route Table](screenshots/routetable.png)

### EC2 Instance Running
![EC2](screenshots/ec2.png)

### Security Group Rules
![Security Group](screenshots/securitygroup.png)

### SSH Connection Confirmed
![SSH](screenshots/ssh.png)

---

## Key Concepts Demonstrated

**CIDR notation** — the VPC owns `10.0.0.0/16` (65,536 addresses). The subnet is a `/24` slice (256 addresses) carved from that range. Subnets must be non-overlapping subdivisions of the parent VPC CIDR block.

**Internet Gateway** — without the IGW the VPC is completely isolated. Creating it is not enough — it must be attached to the VPC and referenced in the route table before traffic can flow to the internet.

**Route table** — the rule `0.0.0.0/0 → igw` tells AWS to send all outbound traffic through the internet gateway. Without this rule the IGW exists but traffic has no path to use it.

**Security groups** — stateful firewalls at the instance level. Port 22 (SSH) was restricted to my public IP only rather than `0.0.0.0/0`. This applies the principle of least privilege at the network level.

**Key pair authentication** — SSH uses asymmetric cryptography. AWS stores the public key on the EC2 instance. The private key (.pem file) is kept locally. The `chmod 400` command restricts the key file permissions so only the owner can read it — SSH refuses to connect with loose permissions as a security measure.

---

## Infrastructure Details

| Component | Value |
|---|---|
| VPC CIDR | `10.0.0.0/16` |
| Subnet CIDR | `10.0.1.0/24` |
| Availability Zone | us-east-2a |
| Instance type | t3.micro |
| AMI | Amazon Linux 2023 |
| SSH port | 22 |
| SSH source | My IP only |

---

## Challenges Encountered and Lessons Learned

### Wrong CIDR range
Initially entered a subnet CIDR from a different IP family than the VPC. The subnet must be a slice of the parent VPC range — if the VPC is `10.0.0.0/16` the subnet must start with `10.0.x.x`. Entering an address from a different family is rejected immediately by AWS.

### Subnet overlap
Hit an overlap error because AWS default subnets already occupied part of the address range. Fixed by checking existing subnets in the VPC console and selecting a non-overlapping block.

### Default VPC conflict
Originally launched the EC2 into the AWS default VPC instead of a custom one. The default VPC already had an IGW attached and AWS only allows one IGW per VPC. Fixed by terminating all resources and rebuilding from scratch using a custom VPC where every component was configured manually.

### Security group open to the internet
Initial setup had SSH open to all sources (`0.0.0.0/0`), meaning any IP on the internet could attempt to connect. Fixed by restricting the inbound rule to My IP only using the AWS auto-detect dropdown.

### Local IP vs public IP
Manually entered the local network IP assigned by the router instead of the public-facing IP. AWS needs the public IP since that is how traffic arrives from outside the home network. The SSH connection was timing out because AWS was expecting traffic from an address that would never arrive. Fixed by using the My IP dropdown rather than typing manually.

### SSH key permissions
SSH refuses to connect if the `.pem` key file has permissions that are too open. Running `chmod 400` on the file restricts access so only the owner can read it — this is a security requirement enforced by the SSH client, not an optional step.

---

## Relation to Project 2

This VPC serves as the deployment infrastructure for the [F1 Pit Lane Dashboard](https://github.com/victoriaalicex/f1-pitlane-dashboard). The EC2 instance hosts the Node.js application, demonstrating end-to-end ownership from network architecture to application deployment.

