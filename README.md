---

# Deploy EC2 and Install Gooseberry with Ansible via GitHub Actions

This project automates the provisioning of an AWS EC2 instance and deploys the [Gooseberry](https://github.com/robanybody1920/gooseberry) application via Docker using Ansible, all triggered through a GitHub Actions workflow.

---

## Overview

With a single manual trigger from GitHub Actions (`workflow_dispatch`), this repository:

1. **Creates an EC2 instance** (Debian-based, `t2.micro`) with a security group.
2. **Installs Docker** and deploys the Gooseberry container.
3. **Exposes Gooseberry** on port `8080` (or `80` if configured).
4. **Prints the access URL** dynamically after deployment.
5. **Updates the MOTD** (Message of the Day) on the EC2 instance.

---

## Repository Structure

```
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions workflow file
├── roles/
│   ├── pro_build/              # Role to create EC2 instance and security group
│   └── pro_gooseberry/         # Role to install and configure Gooseberry
├── pbk_gooseberry.yml             # Main Ansible playbook 
├── motd.j2                     # Template for the EC2 MOTD
└── README.md                   # This file
```

---

## Requirements

### GitHub Secrets
Add these secrets to your repository:

| Secret Name             | Description                                      |
|------------------------|-------------------------------------------------|
| `AWS_ACCESS_KEY_ID`    | AWS access key with EC2 permissions             |
| `AWS_SECRET_ACCESS_KEY`| AWS secret key                                  |
| `SSH_PRIVATE_KEY`      | Base64-encoded SSH key matching `key_name`      |

Encode your private key:
```bash
base64 -w 0 ~/.ssh/ITM350.pem
```

---

## How It Works

### 1. GitHub Actions Workflow (`deploy.yml`)
- Sets up AWS authentication.
- Configures SSH for Ansible.
- Installs Ansible + dependencies (`amazon.aws`, `community.docker`).
- Runs the Ansible playbook.

### 2. Ansible Playbook (`pbk_checkmk.yml`)
- **EC2 Provisioning**:
  - Creates a security group (`disorganized-sg`) with ports `22` (SSH), `80` (HTTP), and `443` (HTTPS) open.
  - Launches a Debian EC2 instance tagged `Name=disorganizer`.
- **Gooseberry Deployment**:
  - Installs Docker and `docker-compose`.
  - Pulls the `robanybody1920/gooseberry:latest` image.
  - Runs the container with port mapping (`8080:8080` by default).

---

## Triggering Deployment

1. Navigate to **Actions** → **Deploy EC2 with Ansible** in your repo.
2. Click **Run workflow**.

---

## Accessing Gooseberry

After deployment, access the app at:  
**`http://<EC2_PUBLIC_DNS>:8080`**  

Example URL:  
`http://ec2-12-34-56-78.compute-1.amazonaws.com:8080`

---

## Variables Reference

| Variable              | Default Value                     | Description                          |
|-----------------------|-----------------------------------|--------------------------------------|
| `instance_name`       | `disorganizer`                    | EC2 instance Name tag                |
| `key_name`            | `ITM350`                          | EC2 key pair name                    |
| `security_group_name` | `disorganized-sg`                 | Security group name                  |
| `instance_type`       | `t2.micro`                        | EC2 instance size                    |
| `region`              | `us-east-1`                       | AWS region                           |
| `image`               | `robanybody1920/gooseberry:latest`| Gooseberry Docker image              |
| `container_name`      | `gooseberry`                      | Docker container name                |
| `web_port`            | `8080`                            | Host port for Gooseberry             |

---

## Teardown

To destroy the EC2 instance:
```bash
aws ec2 terminate-instances --instance-ids <your-instance-id> --region us-east-1
```

---

## Troubleshooting

| Issue                  | Solution                                    |
|------------------------|---------------------------------------------|
| SSH connection failed  | Verify `SSH_PRIVATE_KEY` matches `key_name` |
| Port 8080 unreachable  | Check security group rules                  |
| Container not starting | View logs: `docker logs gooseberry`         |

---
