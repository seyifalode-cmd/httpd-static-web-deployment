# HTTPD Static Web Deployment

Oluwaseyi Michael Falode | Cloud & DevOps Engineering | Toronto, ON

Provisioning an AWS EC2 instance with Terraform and configuring it as an Apache web server using Ansible — fully automated from infrastructure creation to content deployment, with zero manual steps after `terraform apply`.

---

## Project at a Glance

| | |
|---|---|
| **Tools Used** | Terraform · Ansible · AWS EC2 · Apache HTTPD |
| **Platform** | AWS (us-east-1) — Amazon Linux 2 |
| **Languages** | HCL (Terraform) · YAML (Ansible) |
| **What It Does** | Provisions an EC2 instance and deploys a static website using a fully automated Terraform + Ansible pipeline |

## The Problem This Project Solves

Manually setting up a web server is a repeatable but error-prone process. An engineer SSHs into a server, installs packages, uploads files, configures permissions, starts services — and if anything goes wrong, they retrace their steps from memory. When the server needs to be rebuilt (hardware failure, scaling, environment promotion), the entire process repeats with the same risk of human error.

Infrastructure as code eliminates this entirely. Terraform creates the server. Ansible configures it. The website goes live in a single command with no manual intervention. The entire setup is version-controlled, reviewable, and reproducible in any AWS region.

This project demonstrates the Terraform provisioner pattern — using `file` and `remote-exec` provisioners to bridge the gap between infrastructure creation and application configuration, without requiring a separate CI/CD trigger.

## Architecture

```
LOCAL MACHINE
       |
       | terraform apply
       v
TERRAFORM (main.tf)
  ├── Creates EC2 instance (t3.micro, Amazon Linux 2)
  ├── Creates Security Group (SSH :22, HTTP :80)
  ├── Creates Key Pair for SSH access
  │
  ├── Provisioner: file → copies install_httpd.yaml to EC2
  ├── Provisioner: file → copies Einstein.zip to EC2
  └── Provisioner: remote-exec
          |
          | installs Ansible on EC2
          | runs: ansible-playbook install_httpd.yaml
          v
ANSIBLE (install_httpd.yaml)
  ├── Installs Apache httpd via yum
  ├── Unarchives Einstein.zip to /tmp
  ├── Copies index.html + einstein.jpg to /var/www/html/
  └── Starts and enables httpd service
          |
          v
LIVE WEBSITE at http://<EC2-PUBLIC-IP>/
```

## Repository Structure

```
httpd-static-web-deployment/
├── main.tf               # EC2, security group, key pair, provisioners
├── variables.tf          # SSH key path variables
├── outputs.tf            # Outputs EC2 public IP after apply
├── install_httpd.yaml    # Ansible playbook: install httpd, deploy content
├── Einstein.zip          # Static website content (HTML + images)
└── .gitignore
```

## Key Files Explained

### `main.tf` — Infrastructure + Bootstrap

Provisions the EC2 instance and uses three Terraform provisioners to automate the configuration step:
- `file` provisioner copies the Ansible playbook to the server
- `file` provisioner copies the website content archive
- `remote-exec` provisioner installs Ansible and runs the playbook

### `install_httpd.yaml` — Ansible Playbook

```yaml
tasks:
  - install httpd via yum
  - unarchive Einstein.zip to /tmp
  - copy index.html and einstein.jpg to /var/www/html/
  - start and enable httpd service
```

## How to Reproduce

**Prerequisites:** Terraform installed, AWS credentials configured, SSH key pair generated

```bash
# 1. Clone the repository
git clone https://github.com/seyifalode-cmd/httpd-static-web-deployment.git
cd httpd-static-web-deployment

# 2. Set your SSH key paths in variables.tf or via CLI flags
terraform apply \
  -var="ssh_key_private=/path/to/your/key.pem" \
  -var="ssh_key_public=/path/to/your/key.pub"

# 3. Wait for apply to complete — EC2 creates, Ansible runs automatically
# 4. Visit the output public IP in your browser
# 5. Destroy when done
terraform destroy
```

## What This Demonstrates

- End-to-end automation: from blank AWS account to live website in one command
- Terraform provisioner pattern for bootstrapping instances without external CI/CD
- Ansible idempotent configuration management
- Separation of concerns: Terraform owns infrastructure, Ansible owns configuration

---

*Oluwaseyi Michael Falode · Cybersecurity & Cloud Security Engineer · Toronto, ON*
