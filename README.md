
# MongoDB Atlas Ansible Role

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
```

### 2. Install requirements
Make sure you have:
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [Terraform](https://developer.hashicorp.com/terraform/install)
- Python packages for Ansible collections if required.

You can install collections (example):
```bash
ansible-galaxy collection install community.general
```

---

### 3. Define your variables
Edit your `group_vars` or `vars` file with required parameters:

```yaml
atlas_project_id: "YOUR_PROJECT_ID"
cluster_name: "test-cluster"
provider_name: "AWS"
region_name: "US_EAST_1"
cluster_instance_size: "M10"
ip_whitelist:
  - "0.0.0.0/0"

db_user:
  username: "app_user"
  password: "securepassword"
  database_name: "app_db"
  roles:
    - roleName: "readWrite"
      databaseName: "app_db"
```

---

### 4. Run the playbook
```bash
ansible-playbook create-atlas-cluster.yml
```

The playbook will:
1. Render the Terraform templates with your variables.
2. Initialize and apply Terraform to create the MongoDB Atlas cluster.

---

## 📂 Repository structure
```
.
├── roles/
│   └── terraform/
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           ├── main.tf.j2
│           └── terraform.tfvars.j2
├── vars/
│   └── main.yml
├── create-atlas-cluster.yml
└── README.md
```

---

## ✅ Requirements
- Ansible >= 2.9
- Terraform >= 1.0
- MongoDB Atlas API keys or credentials set as environment variables or in your vars.

---

## 🚧 Future enhancements
- Support for additional MongoDB Atlas resources (projects, alert configurations, etc.)
- Automated destroy playbook to clean up infrastructure.
- Molecule tests for Ansible role.

---

## 📜 License
MIT

---

## 🤝 Contributing
PRs and issues are welcome! Please open an issue first to discuss changes or new features.

---

## 📬 Contact
Created by [Your Name](https://github.com/yourusername) - feel free to reach out!

---

## ✍️ Example Jinja2 templates

### `main.tf.j2`
```jinja
terraform {
  required_providers {
    mongodbatlas = {
      source  = "mongodb/mongodbatlas"
      version = "~> 1.9.0"
    }
  }
}

provider "mongodbatlas" {
  public_key  = var.atlas_public_key
  private_key = var.atlas_private_key
}

resource "mongodbatlas_cluster" "cluster" {
  project_id   = var.atlas_project_id
  name         = var.cluster_name
  cluster_type = "REPLICASET"

  provider_name = var.provider_name
  provider_region_name = var.region_name
  provider_instance_size_name = var.cluster_instance_size
}

resource "mongodbatlas_database_user" "db_user" {
  username = var.db_username
  password = var.db_password
  project_id = var.atlas_project_id
  roles {
    role_name     = var.db_role_name
    database_name = var.db_database_name
  }
}

resource "mongodbatlas_project_ip_whitelist" "ips" {
  count      = length(var.ip_whitelist)
  project_id = var.atlas_project_id
  cidr_block = element(var.ip_whitelist, count.index)
}
```

---

### `terraform.tfvars.j2`
```jinja
atlas_project_id       = "{{ atlas_project_id }}"
cluster_name           = "{{ cluster_name }}"
provider_name          = "{{ provider_name }}"
region_name            = "{{ region_name }}"
cluster_instance_size  = "{{ cluster_instance_size }}"

db_username            = "{{ db_user.username }}"
db_password            = "{{ db_user.password }}"
db_database_name       = "{{ db_user.database_name }}"
db_role_name           = "{{ db_user.roles[0].roleName }}"

ip_whitelist           = [ {% for ip in ip_whitelist %}"{{ ip }}"{% if not loop.last %}, {% endif %}{% endfor %} ]

atlas_public_key       = "{{ atlas_public_key }}"
atlas_private_key      = "{{ atlas_private_key }}"
```
