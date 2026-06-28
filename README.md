# Terraform AWS Web Infrastructure

![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazonaws&logoColor=white)
![EC2](https://img.shields.io/badge/Amazon%20EC2-FF9900?style=flat&logo=amazonec2&logoColor=white)
![S3](https://img.shields.io/badge/Amazon%20S3-569A31?style=flat&logo=amazons3&logoColor=white)
![Status](https://img.shields.io/badge/Status-Active-success?style=flat)
![IaC](https://img.shields.io/badge/IaC-Terraform-7B42BC?style=flat)

Provision a highly available AWS web infrastructure using Terraform. Deploys two Ubuntu EC2 instances across separate Availability Zones behind a Application Load Balancer, within a custom VPC — all defined as code.

---

## Table of Contents

- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Quick Start](#quick-start)
- [Resources](#resources)
- [Configuration](#configuration)
- [Security](#security)
- [Outputs](#outputs)
- [Cleanup](#cleanup)

---

## Architecture

```
                              ┌─────────────────────┐
                              │       Internet      │
                              └──────────┬──────────┘
                                         │
                              ┌──────────▼──────────┐
                              │   Internet Gateway  │
                              └──────────┬──────────┘
                                         │
                    ┌────────────────────────────────────────┐
                    │              VPC  10.0.0.0/16          │
                    │                                        │
                    │         ┌──────────────────┐           │
                    │         │   Route Table    │           │
                    │         │ 0.0.0.0/0 → IGW  │           │
                    │         └──────────────────┘           │
                    │                                        │
                    │         ┌──────────────────┐           │
                    │         │    AL Balancer   │           │
                    │         │   (Application)  │           │
                    │         └────────┬─────────┘           │
                    │                  │                     │
                    │         ┌────────┴─────────┐           │
                    │         │                  │           │
                    │  ┌──────▼───────┐  ┌───────▼──────┐    │
                    │  │  us-east-1a  │  │  us-east-1b  │    │
                    │  │ 10.0.1.0/24  │  │ 10.0.2.0/24  │    │
                    │  │              │  │              │    │
                    │  │  WebServer-1 │  │  WebServer-2 │    │
                    │  │   t3.micro   │  │   t3.micro   │    │
                    │  └──────────────┘  └──────────────┘    │
                    │                                        │
                    └────────────────────────────────────────┘
```

**Traffic Flow**

```
User → Internet Gateway → Route Table → Load Balancer → EC2 (Round Robin)
```

---

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| Terraform | >= 1.0 | [terraform.io](https://terraform.io) |
| AWS CLI | >= 2.0 | [aws.amazon.com/cli](https://aws.amazon.com/cli) |
| AWS Account | - | [aws.amazon.com](https://aws.amazon.com) |

Configure AWS credentials before running:

```bash
aws configure
```

---

## Quick Start

```bash
# Clone repository
git clone https://github.com/devaasirvathamsj/terraform-aws-web-infrastructure.git
cd terraform-aws-web-infrastructure

# Initialize Terraform
terraform init

# Review execution plan
terraform plan

# Deploy infrastructure
terraform apply
```

After apply completes, the ELB DNS endpoint will be printed as output.

---

## Resources

The following AWS resources are provisioned:

| Resource | Count | Details |
|---|---|---|
| VPC | 1 | CIDR: `var.cidr` |
| Public Subnet | 2 | us-east-1a, us-east-1b |
| Internet Gateway | 1 | Attached to VPC |
| Route Table | 1 | Public — routes to IGW |
| Security Group | 1 | HTTP (80), SSH (22) |
| EC2 Instance | 2 | t3.micro, Ubuntu 24.04 LTS |
| Application Load Balancer | 1 | Internet-facing, HTTP:80 |
| Target Group | 1 | HTTP:80 with health checks |
| S3 Bucket | 1 | Public read — static assets |

---

## Configuration

### Variables

| Variable | Default | Description |
|---|---|---|
| `cidr` | - | VPC CIDR block (e.g. `10.0.0.0/16`) |

Define variables in `terraform.tfvars`:

```hcl
cidr = "10.0.0.0/16"
```

### User Data

Each EC2 instance runs a bootstrap script on first launch:

| Script | Instance | Purpose |
|---|---|---|
| `user_data_1.sh` | WebServer-1 | Install and configure web server |
| `user_data_2.sh` | WebServer-2 | Install and configure web server |

### Health Check

The load balancer performs health checks on each instance:

```
Protocol  : HTTP
Path      : /
Interval  : 30 seconds
Timeout   : 5 seconds
Threshold : 2 consecutive checks
```

---

## Project Structure

```
terraform-aws-web-infrastructure/
├── main.tf               # Resource definitions
├── variable.tf           # Input variable declarations
├── outputs.tf            # Output value definitions
├── terraform.tfvars      # Variable values (not committed)
├── user_data_1.sh        # WebServer-1 bootstrap script
├── user_data_2.sh        # WebServer-2 bootstrap script
└── README.md
```

---

## Security

### Security Group Rules

| Direction | Protocol | Port | Source | Purpose |
|---|---|---|---|---|
| Inbound | TCP | 80 | `0.0.0.0/0` | HTTP traffic |
| Inbound | TCP | 22 | `0.0.0.0/0` | SSH access |
| Outbound | All | All | `0.0.0.0/0` | Unrestricted |

> **Warning:** SSH is open to all IP addresses in this configuration. For production environments, restrict access to a known CIDR block.

```hcl
cidr_blocks = ["YOUR_IP/32"]
```

---

## Outputs

| Output | Description |
|---|---|
| `elb_dns_name` | DNS name of the Classic Load Balancer |

After a successful apply:

```bash
terraform output elb_dns_name
# my-elb-800681217.us-east-1.elb.amazonaws.com
```

Open in browser to verify the deployment. Refreshing the page will round-robin between WebServer-1 and WebServer-2, confirming load balancing is active.

---

## Screenshots

**EC2 Instances**
![EC2 Instances](./screenshots/ec2-instances.png)

**Load Balancer**
![Load Balancer](./screenshots/load-balancer.png)

**Terraform Apply**
![Terraform Apply](./screenshots/terraform-apply.png)

**Application**
![Application](./screenshots/app.png)

---

## Cleanup

Destroy all provisioned resources to avoid ongoing charges:

```bash
terraform destroy
```

---

## Author

**Deva Asirvatham SJ**

[![GitHub](https://img.shields.io/badge/GitHub-devaasirvathamsj-181717?style=flat&logo=github)](https://github.com/devaasirvathamsj)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat&logo=linkedin)](https://linkedin.com/in/devaasirvathamsj)

---

## License

MIT License. See [LICENSE](./LICENSE) for details.
