#  Deploy EC2 and Install Checkmk with Ansible via GitHub Actions

This project automates the provisioning of an AWS EC2 instance and installs [Checkmk](https://checkmk.com/) monitoring via Docker using Ansible, all triggered through a GitHub Actions workflow.

##  Overview

With a single manual trigger from GitHub Actions (`workflow_dispatch`), this repository:

1. Checks if a named EC2 instance exists.
2. Creates a security group if necessary.
3. Launches a new Debian-based EC2 instance using the latest AMI.
4. Connects to the instance via SSH.
5. Installs Docker and Checkmk via Docker container.
6. Sets the Checkmk admin password.
7. Automatically configures your MOTD.
8. Runs entirely through Ansible playbooks for reproducibility and clarity.


---

##  Repository Structure

 ├── .github/
 │ └── workflows/
 │ └── deploy.yml # GitHub Actions workflow file
 ├── roles/
 │ ├── pro_build/ # Role to create EC2 instance and security group 
 │ └── pro_checkmk/ # Role to install and configure Checkmk 
 ├── pbk_checkmk.yml # Entry point Ansible playbook 
 ├── motd.j2 # Template for the EC2 Message of the Day 
 └── README.md # This file

---

##  Requirements

Before running, ensure you’ve added the following secrets to your GitHub repository:

| Secret Name             | Description                                      |
|------------------------|---------------------------------------------------|
| `AWS_ACCESS_KEY_ID`    | AWS access key with EC2 and IAM permissions       |
| `AWS_SECRET_ACCESS_KEY`| Secret key associated with the AWS access key     |
| `AWS_SESSION_TOKEN`    | (Optional) Temporary session token                |
| `SSH_PRIVATE_KEY`      | Base64-encoded SSH private key matching `key_name`|

> You can encode your private key like this:
> ```bash
> base64 -w 0 ~/.ssh/id_rsa
> ```

---

##  How It Works

### GitHub Actions Workflow (`deploy.yml`)
This workflow:

- Authenticates with AWS using your secrets.
- Sets up an SSH key on the runner.
- Scans and registers the EC2 host to `known_hosts`.
- Installs Python, Ansible, and required collections (`amazon.aws`, `community.docker`).
- Executes the Ansible playbook `pbk_checkmk.yml`.

### Ansible Playbook: `pbk_checkmk.yml`

The playbook handles:

1. **Security Group Management**
   - Checks for an existing group called `disorganized-sg`.
   - If not found, creates one with common ports (`22`, `80`, `443`, etc.).

2. **EC2 Instance Provisioning**
   - Looks for an instance tagged `Name=disorganizer`.
   - If not found, launches a new `t2.micro` instance using the latest Debian 11 AMI.
   - Associates the instance with the defined key and security group.

3. **Instance Preparation**
   - Gathers instance metadata.
   - Adds the instance to a temporary dynamic Ansible inventory.

4. **Checkmk Deployment**
   - Installs `docker.io` and `docker-compose`.
   - Pulls the `checkmk` Docker image.
   - Starts the container and maps web (8080) and agent (8000) ports.
   - Sets the Checkmk admin password (`swordfish` by default).

---

##  Triggering the Deployment

Once everything is configured:

1. Go to the **Actions** tab in your GitHub repository.
2. Select the workflow: `Deploy EC2 with Ansible`.
3. Click **Run workflow**.


---

##  Accessing Checkmk

Once complete, visit:
http://<EC2_PUBLIC_IP>:8080


Login with:
- **Username**: `cmkadmin`
- **Password**: `swordfish`


---

##  Variables Reference

These can be found in the playbook but are summarized here:

| Variable Name         | Default Value     | Description                           |
|-----------------------|-------------------|---------------------------------------|
| `instance_name`       | `disorganizer`    | EC2 instance Name tag                 |
| `key_name`            | `ITM350`          | Existing EC2 key pair                 |
| `security_group_name` | `disorganized-sg` | Security group name                   |
| `instance_type`       | `t2.micro`        | EC2 instance size                     |
| `region`              | `us-east-1`       | AWS region                            |
| `checkmk_image`       | `checkmk/check-mk-raw:2.2.0-latest` | Docker image to use |
| `container_name`      | `checkmk_monitoring` | Name of the Docker container       |
| `web_port`            | `8080`            | Host port for Checkmk web UI          |
| `agent_port`          | `8000`            | Host port for Checkmk agent           |

---

## Teardown

To terminate your EC2 instance manually:

```bash
aws ec2 terminate-instances --instance-ids <instance-id> --region us-east-1


Troubleshooting
SSH failures? Double-check the key used matches ITM350.

EC2 instance not found? Ensure the Name tag is exactly disorganizer.

Checkmk unreachable? Make sure port 8080 is open and the EC2 instance is running.
