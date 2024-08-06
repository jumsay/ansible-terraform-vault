
# Automate Terraform with Ansible Playbook and vault
How to automate Terraform provisionning with Ansible playbook

![Ansible](https://img.shields.io/badge/Ansible-required-blue?logo=ansible&logoColor=white&logoSize=auto) 
![Terraform](https://img.shields.io/badge/Terraform-required-blue?logo=terraform&logoColor=white&logoSize=auto) 
![Amazon](https://img.shields.io/badge/amazon_EC2-required-blue?logo=amazon&logoColor=white&logoSize=auto)

## Features
- Automates Terraform provisioning with Ansible
- Uses Ansible Vault to securely handle sensitive information

## Requirements
1. Install Terraform
2. Install Ansible
3. Create key-pairs on aws

## Getting Started
The provided instructions detail a comprehensive approach to automating the provisioning of AWS instances using Terraform, Ansible, and Ansible Vault. I'll summarize and provide a step-by-step guide along with the necessary scripts to streamline your setup.
You can clone this repository before

### Directory structure

Ensure the following directory structure in your project:

```bash
project/
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── provider.tf
├── ansible/
│   ├── hosts
│   ├── playbook.yml
│   ├── group_vars/
│   │   ├── global_vars.yml
│   │   └── vars.yml
│   └── roles/
│       └── terraform-pro/
│           ├── tasks/
│               ├── terraform_pro.yml
│               ├── get_instances_ip.yml
│               └── instances_update.yml
```


### Step 1: Create terraform Configuration Files

```sh
mkdir -p ~/<project folder>/terraform
```


Inside the folder create a simple terraform main.tf
```sh title="main.tf"
nano  ~/<project folder>/terraform/main.tf
```

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "<instance name>" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleInstance"
  }

  provisioner "local-exec" {
    command = "echo ${self.public_ip} >> ../ansible/hosts"
  }
}

```
Create variables.tf
```sh
nano  ~/<project folder>/terraform/variables.tf
```

```hcl
variable "aws_access_key" {
  description = "AWS access key"
}

variable "aws_secret_key" {
  description = "AWS secret key"
}

```
provider.tf
```sh
nano  ~/<project folder>/terraform/provider.tf
```

```hcl
terraform {
  required_version = ">= 0.13"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 5"
    }
  }
}

provider "aws" {
  region     = "us-east-1"
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
}
```

outputs.tf
```sh
nano  ~/<project folder>/terraform/outputs.tf
```
```hcl
output "instance_public_ip" {
  value = aws_instance.example.public_ip
}
```

### Step 2: Create Ansible Variables and Vault


```sh
mkdir -p ~/<project folder>/group_vars
```

create vars.yml file witch will contain the variable need by terraform to 
connecte on aws.

```sh
nano  ~/<project folder>/group_vars/vars.yml
```

```yaml
public_key: “ssh-rsa AAAXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX” 
aws_access_key_id: AKXXXXXXXXXXXXXXXX
aws_secret_access_key: 9iVmdMsLXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```
####  Encypt variables with Ansible Vault

```sh
openssl rand -base64 2048 > pass.vault
ansible-vault create group_vars/vars.yml --vault-password-file vars.vault
```
After this if you need to edit the variables, replace "create" by "edit" in the command


Create global variables for the project global_vars.yml

```sh
nano  ~/<project folder>/group_vars/global_vars.yml
```
```yaml
terraform_dir: "~/<project folder>/terraform/"
```


### Step 3: Create Ansible Roles
We will create a folder where we will keep our roles.
```sh
mkdir -p ~/<project folder>/ansible/roles
```
- Create inventory file

Create a file which will recieve ip address for instances provisionning by terraform
```sh
nano ~/<project folder>/ansible/hosts
[localhost]
127.0.0.1
```

We will then move to this folder:
```sh
cd  ~/<project folder>/ansible/roles
```
#### create terraform provisionning role using Ansible Galaxy
```sh
ansible-galaxy init terraform-pro
```
- Create the terraform provisionning role tasks

go to <project folder>/ansible/roles/terraform-pro/tasks/

create file terraform_pro.yml

```sh
nano  ~/<project folder>/roles/terraform-pro/tasks/terraform_pro.yml
```

```yaml
---
# Provisionning infrastructure with Terraform
- name: terraform provisionning
  shell: |
    chdir="{{ terraform_dir }}"
    terraform init
    terraform validate
    terraform plan
    terraform apply --auto-approuve
```

- Get Instance IP create file get_instances_ip.yml
```sh
nano  ~/<project folder>/roles/terraform-pro/tasks/terraform_ip.yml
```
```yaml
---
- name: Get instance aws ips
  shell: |
    chdir="<project folder>/terraform/"
    terraform output -raw instance_public_ip
  register: instance_public_ip

- name: Sotre instance in host file
  shell: |
    echo '[instances]' >> ~/<project folder>/ansible/hosts
    echo instance_public_ip >> ~/<project folder>/ansible/hosts
```
- Update instances create file instances_update.yml

```sh
nano  ~/<project folder>/roles/terraform-pro/tasks/instances_update.yml
```

```yaml
---
# Update all debian or ubuntu instance 
- name: update instances 
    ansible.builtin.apt:
    name: '*'
    state: latest

- name : Update all packages
  ansible.builtin.apt:
    update_cache: true
    
```


### Step 4: Create Ansible PlayBook to use the role


```sh
nano  ~/<project folder>playbook.yml
```

```yaml
---
- name: Provisionning instances with Terraform
  hosts: [localhost]
  connection: local
  vars_files:
    - "group_vars/global_vars.yml"
    - "group_vars/vars.yml"
  tasks:
    - name:
      included_tasks: roles/terraform-pro/tasks/terraform_pro.yml

    - name:
      included_tasks: roles/terraform-pro/tasks/get_instances_ip.yml

- name: Update all instances
  hosts: [instances]
  vars_files:
    - "group_vars/vars.yml"
  tasks:
    name:
    included_tasks: roles/terraform-pro/tasks/instances_update.yml

```

### Step 5: Run the Playbook

```sh
ansible-playbook -i ~/project/ansible/host ~/project/ansible/playbook.yml --vault-password-file ~/project/ansible/vars.vault
```

By following these steps, you will have an automated workflow for provisioning AWS instances with Terraform, securing sensitive data with Ansible Vault, and managing infrastructure with Ansible playbooks.
