# Terraform AWS Singapore — EC2 + VPC Deployment

## Project Summary
Hands-on Infrastructure as Code (IaC) project deploying real AWS infrastructure 
in Singapore (ap-southeast-1) using Terraform. Completed the full HashiCorp 
Getting Started with AWS tutorial, adapted for the Singapore region.

---

## What Was Built
- Amazon EC2 (Elastic Compute Cloud) instance running Ubuntu 22.04 on t3.micro
- Custom VPC (Virtual Private Cloud) with public and private subnets
- Internet Gateway, Route Tables, and Security Groups
- Full lifecycle: init → plan → apply → manage → destroy

---

## Technologies Used
| Tool | Purpose |
|---|---|
| Terraform v1.15.1 | Infrastructure as Code (IaC) |
| AWS CLI (Amazon Web Services Command Line Interface) | Authentication and resource querying |
| AWS EC2 | Virtual machine compute |
| AWS VPC | Network isolation |
| HCL (HashiCorp Configuration Language) | Configuration syntax |
| Visual Studio Code | Code editor |
| Git + GitHub | Version control and documentation |

---

## Architecture
```
terraform.tf          → Provider config (HashiCorp/AWS, Singapore region)
variables.tf          → Input variables (instance name, instance type)
main.tf               → EC2 instance inside custom VPC
output.tf             → Outputs instance private DNS hostname
```
AWS ap-southeast-1 (Singapore)
└── VPC: JunRong-vpc (10.0.0.0/16)
├── Public Subnet:  10.0.101.0/24 (ap-southeast-1a)
├── Private Subnet: 10.0.1.0/24  (ap-southeast-1b)
├── Private Subnet: 10.0.2.0/24  (ap-southeast-1c)
├── Internet Gateway
├── Route Tables
└── EC2: t3.micro Ubuntu 22.04 (Totoro-learn-terraform)
---

## Files
| File | Purpose |
|---|---|
| `terraform.tf` | Terraform and AWS provider configuration |
| `main.tf` | EC2 instance and VPC module resources |
| `variables.tf` | Input variables for instance name and type |
| `output.tf` | Output values exposing instance hostname |
| `.gitignore` | Prevents state files and secrets from being pushed |
| `screenshots/` | Evidence of real deployment |

---

## Real Challenges Faced and How I Solved Them

### 1. Terraform Not Recognised in PowerShell
**Problem:** After downloading terraform.exe, running `terraform --version` 
failed with CommandNotFoundException.  
**Cause:** `terraform.exe` was placed in `C:\terraform` but PATH had a typo — 
`C:\terrform` (missing letter "a").  
**Fix:** Corrected the typo in System Environment Variables PATH, 
reopened PowerShell.

### 2. Project Folder Created in Wrong Location
**Problem:** Ran `mkdir` while PowerShell was in `C:\Windows\System32\`, 
creating the project inside a protected system folder. VS Code could not 
write files there due to permission errors.  
**Fix:** Recreated the project folder under `C:\Users\Totoro\` where 
user has full write permissions.

### 3. AMI (Amazon Machine Image) Not Found in Singapore Region
**Problem:** HashiCorp tutorial uses `us-west-2` (Oregon). The AMI filter 
patterns in the tutorial did not match Singapore region naming conventions, 
returning null results.  
**Fix:** Used AWS CLI to directly query the correct AMI for ap-southeast-1:
```bash
aws ec2 describe-images --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
  --query "sort_by(Images, &CreationDate)[-1].[ImageId,Name]" \
  --region ap-southeast-1
```
Result: `ami-03d73333e3a64fd03` (Ubuntu 22.04 LTS Jammy Jellyfish)

### 4. Instance Type Not Free Tier Eligible
**Problem:** Tutorial uses `t2.micro` but this instance type is not free 
tier eligible on newer AWS accounts in Singapore.  
**Fix:** Queried AWS CLI to find free tier eligible instance types:
```bash
aws ec2 describe-instance-types \
  --filters "Name=free-tier-eligible,Values=true" \
  --query "InstanceTypes[*].InstanceType" \
  --region ap-southeast-1
```
Result: Used `t3.micro` instead — the modern free tier replacement.

### 5. HCL Variable Syntax Error
**Problem:** Variables were written as `"var.instance_type"` (with quotes), 
causing Terraform to treat them as literal strings instead of variable 
references.  
**Fix:** Removed quotes — correct HCL syntax is `var.instance_type` 
without quotes.

### 6. EC2 Instance Replaced When Adding VPC
**Problem:** After adding the VPC module, Terraform destroyed the existing 
EC2 instance and created a new one. This was unexpected.  
**Understanding:** AWS does not support moving an existing EC2 instance into 
a new VPC. Terraform correctly planned a destroy-and-replace (-/+) operation. 
This is expected behaviour, not an error.

---

## Key Terraform Commands Used
```bash
terraform init      # Download providers and modules
terraform plan      # Preview changes before applying
terraform apply     # Deploy infrastructure to AWS
terraform output    # View output values
terraform state list # List all managed resources
terraform destroy   # Remove all infrastructure
```

---

## Screenshots
| Screenshot | What It Shows |
|---|---|
| `terraform_innit.png` | Successful terraform init |
| `terraform_apply.png` | First EC2 deployment |
| `totoro_EC2.png` | Live EC2 instance in AWS Singapore console |
| `terraform_var.png` | variables.tf configuration |
| `terraform_output.png` | output.tf configuration |
| `terraform_module_var.png` | VPC module deployment with 16 resources |
| `EC2_replaced.png` | EC2 destroy-and-replace when VPC was added |

---

## What I Learned
- Terraform separates **provider configuration** (terraform.tf) from 
  **resource definitions** (main.tf) for clean project structure
- AMI IDs are **region-specific** — always query for the correct region
- Free tier eligibility is **account-specific** — always verify before assuming
- Terraform **state files** track real infrastructure and must never be 
  committed to GitHub
- Variables in HCL must **never be quoted** — quotes mean literal strings
- Terraform plans a **dependency graph** — resources are created and 
  destroyed in the correct order automatically

---

## Resume Summary
> Deployed AWS infrastructure using Terraform Infrastructure as Code (IaC) 
> including EC2 instances and VPC networking in Singapore (ap-southeast-1). 
> Hands-on experience with terraform init, plan, apply, and destroy lifecycle. 
> Troubleshot region-specific AMI errors, free tier instance compatibility, 
> and HCL variable syntax issues. Documented full deployment on GitHub.
