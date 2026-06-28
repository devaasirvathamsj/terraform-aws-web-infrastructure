# 🚀 Terraform AWS Infrastructure Project

A complete AWS infrastructure built using **Terraform (IaC)** — provisioning a highly available web application with VPC, EC2 instances across multiple Availability Zones, and a Classic Load Balancer.

---

## 📐 Architecture Diagram

```
                         Internet
                            │
                       Route 53 (DNS)
                            │
                    Internet Gateway (IGW)
                            │
              ┌─────────────────────────────┐
              │         VPC (10.0.0.0/16)   │
              │                             │
              │  ┌──────────────────────┐   │
              │  │  Route Table         │   │
              │  │  0.0.0.0/0 → IGW     │   │
              │  └──────────────────────┘   │
              │                             │
              │  ┌──────────────────────┐   │
              │  │  Security Group      │   │
              │  │  Port 80 (HTTP) ✅   │   │
              │  │  Port 22 (SSH)  ✅   │   │
              │  └──────────────────────┘   │
              │                             │
              │  ┌─────────────────────┐    │
              │  │  Classic Load       │    │
              │  │  Balancer (ELB)     │    │
              │  └────────┬────────────┘    │
              │           │                 │
              │    ┌──────┴──────┐          │
              │    │             │          │
              │ ┌──▼──────┐ ┌───▼─────┐    │
              │ │Subnet 1 │ │Subnet 2 │    │
              │ │10.0.1.0 │ │10.0.2.0 │    │
              │ │/24      │ │/24      │    │
              │ │us-east  │ │us-east  │    │
              │ │-1a      │ │-1b      │    │
              │ │         │ │         │    │
              │ │WebServer│ │WebServer│    │
              │ │    1    │ │    2    │    │
              │ │t3.micro │ │t3.micro │    │
              │ └─────────┘ └─────────┘    │
              │                             │
              │  ┌──────────────────────┐   │
              │  │  S3 Bucket           │   │
              │  │  deva-devops-2022    │   │
              │  └──────────────────────┘   │
              └─────────────────────────────┘
```

---

## 🛠️ What I Built

| Resource | Details |
|---|---|
| **VPC** | Custom VPC with CIDR `10.0.0.0/16` |
| **Subnets** | 2 Public subnets across `us-east-1a` and `us-east-1b` |
| **Internet Gateway** | Allows internet access to public subnets |
| **Route Table** | Routes `0.0.0.0/0` traffic to IGW |
| **Security Group** | Allows HTTP (80) and SSH (22) inbound traffic |
| **EC2 Instances** | 2 x `t3.micro` Ubuntu web servers |
| **Classic Load Balancer** | Distributes traffic between both EC2 instances |
| **Target Group** | Health checks on port 80 |
| **S3 Bucket** | Public S3 bucket for static assets |

---

## 🧰 Tech Stack

- **Cloud Provider** → AWS (Amazon Web Services)
- **IaC Tool** → Terraform `v6.52.0`
- **OS** → Ubuntu 24.04 LTS
- **Instance Type** → t3.micro
- **Region** → us-east-1 (N. Virginia)

---

## 📁 Project Structure

```
terraform-project/
├── main.tf              # Main infrastructure code
├── variable.tf          # Input variables
├── outputs.tf           # Output values (ELB DNS)
├── user_data_1.sh       # WebServer-1 bootstrap script
├── user_data_2.sh       # WebServer-2 bootstrap script
└── README.md            # Project documentation
```

---

## ⚙️ How to Run

### Prerequisites
- AWS CLI configured (`aws configure`)
- Terraform installed (`terraform -v`)
- AWS account with required permissions

### Steps

```bash
# Step 1 - Clone the repository
git clone https://github.com/devaasirvatham/terraform-aws-project.git
cd terraform-aws-project

# Step 2 - Initialize Terraform
terraform init

# Step 3 - Format and validate
terraform fmt
terraform validate

# Step 4 - Preview changes
terraform plan

# Step 5 - Apply infrastructure
terraform apply

# Step 6 - Destroy when done (avoid bill!)
terraform destroy
```

---

## 📤 Output

After `terraform apply`, you will get:

```bash
Outputs:
elb_dns_name = "my-elb-xxxxxxx.us-east-1.elb.amazonaws.com"
```

Open the ELB DNS in your browser — traffic will be distributed between:
- **WebServer-1** → `"Welcome devaasirvatham"`
- **WebServer-2** → `"Welcome abishaa"`

Refresh the page to see load balancing in action! ⚡

---

## 🔐 Security

| Rule | Port | Protocol | Source |
|---|---|---|---|
| HTTP Inbound | 80 | TCP | 0.0.0.0/0 |
| SSH Inbound | 22 | TCP | 0.0.0.0/0 |
| All Outbound | All | All | 0.0.0.0/0 |

> ⚠️ **Note:** SSH is open to all IPs for demo purposes.  
> In production, restrict SSH to your IP only!

---

## 💡 Key Learnings

- Infrastructure as Code (IaC) using Terraform
- AWS VPC design with public subnets across multiple AZs
- High availability using multi-AZ EC2 deployment
- Load balancing traffic with Classic ELB
- EC2 bootstrapping using User Data scripts
- S3 bucket configuration with public access

---

## 📸 Screenshots

| Resource | Status |
|---|---|
| EC2 Instances | ✅ Running |
| Load Balancer | ✅ Active |
| Target Group | ✅ Healthy |
| Terraform Apply | ✅ Success |

---

## 🧹 Cleanup

```bash
terraform destroy
```

> Always destroy resources after practice to avoid AWS charges! 💸

---

## 👨‍💻 Author

**Deva Asirvatham**  
AWS DevOps Engineer (Learning)  
📧 [LinkedIn](https://linkedin.com/in/devaasirvatham)  
🐙 [GitHub](https://github.com/devaasirvatham)

---

## 📜 License

MIT License — feel free to use and modify!

---

> *"Infrastructure as Code — Build once, deploy anywhere!"* 🚀
