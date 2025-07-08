# Ansible Terraform MongoDB Atlas

This repository contains an Ansible playbook and role that dynamically generates Terraform configuration files using Jinja2 templates, and then applies them to provision MongoDB Atlas clusters. 

It enables you to use Ansible variables (inventory, extra_vars, etc.) to control your MongoDB Atlas infrastructure, combining the flexibility of Ansible with the power of Terraform for managing cloud resources.

---

## 🚀 Features

- Generate Terraform `main.tf` and `terraform.tfvars` dynamically from Ansible variables.
- Use Jinja2 templating for flexibility and reusability.
- Supports creating MongoDB Atlas clusters with parameters like project ID, cluster specs, database users, and IP access lists.
- Automate Terraform init, plan, and apply from Ansible.
- Example inventory and variable files included.

---

## 🛠 Usage

### 1. Clone the repository
```bash
git clone https://github.com/yourusername/ansible-terraform-mongodb-atlas.git
cd ansible-terraform-mongodb-atlas
