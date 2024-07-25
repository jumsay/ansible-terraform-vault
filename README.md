# Automate Terraform with Ansible Playbook an vault
How to automate Terraform provisionning with Ansible playbook

![Static Badge](https://img.shields.io/badge/Ansible-required-blue?logo=ansible&logoColor=white&logoSize=auto) ![Static Badge](https://img.shields.io/badge/Terraform-required-blue?logo=terraform&logoColor=white&logoSize=auto)

## Features

## Requirements
1. Install Terraform
2. Install Ansible
3. Create key-pairs on aws

## Let's started
you can clone this repository before

### Directory structure
```bash
 project/
├── terraform/
│   ├── main.tf
│   └── variables.tf
└── ansible/
    ├── hosts
    └── playbook.yml
```
### Step 1: Create terraform folder in the project folder
### Step 2: create file in the folder
Create main.tf
Create variables.tf
create output.tf

### Step 3: vars folder
vars.yml
vars.vault (Add .vault in .gitignore file on your project folder

### Step 4: Encypt variables with Ansible Vault

### Step 5: Create the Ansible role folder
We will create a folder where we will keep our roles.
```sh
mkdir -p ~/<project folder>/roles
```
### Step 6: Create inventory folder and add empty hosts file
```sh
mkdir -p ~/<project folder>/inventory
```
Create host file 
which will recieve ip address for instances provisionning by terraform
```sh
touch  ~/<project folder>/inventory/hosts
```

We will then move to this folder:
```sh
cd  ~/ansible/roles
```
### Step 7: create terraform provisionning role using Ansible Galaxy
```sh
ansible-galaxy init terraform-pro
```
### Step 8: Create the terraform provisionning role tasks

### Step 9: Create Ansible PlayBook to use the role

### Step 10: Run the Playbook





