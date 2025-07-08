
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

#### Programs and Packages 
Make sure you have:
- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [Terraform](https://developer.hashicorp.com/terraform/install)
- Python packages for Ansible collections if required.

#### Install Ansible Collections

##### Via the requirements.yml file
If you have only installed ansible-core, be sure to require the following collections in your `requirements.yml` file in the same directory as your playbook:

```
# requirements.yml
collections:
  - name: community.mongodb
    version: 1.7.10
  - name: community.general
    version: 11.0.0
  - name: ansible.posix
    version: 2.0.0
```

You can then run the following command:
`ansible-galaxy collection install -r requirements.yml`

##### Individual collection installation

Alternatively, you install them manually with the following commands:

```
ansible-galaxy collection install community.mongodb:1.7.10
ansible-galaxy collection install community.general:10.0.0
ansible-galaxy collection install ansible.posix:2.0.0
```

---

### 3. Define your variables
Edit your `group_vars` or `vars` file with required parameters:

```yaml
## The value of the `Organization ID` field in the Organization Settings GUI page in
## MongoDB Atlas. This represents the existing Atlas Organization that the Project is a part of.
atlas_org_id: 1234567890abcdef12345678

## The atlas_project_name is the name in the `Project Name` field when looking at the
## Project Settings GUI page in MongoDB Atlas. If there are spaces in the name, use quotes
## when populating this variable. 
atlas_project_name: "Test Project"

### The atlas_project_name is the name in the `Project ID` field when looking at the
## Project Settings GUI page in MongoDB Atlas.
atlas_project_id: 1234567890abcdef12345679

## This is the the desired name of the cluster, and will also be incorporated into the
## database access usernames that will be generated.
atlas_cluster_name: test-cluster

## The desired instance type of replica set members in your cluster
## It can range from M10 to M80.
atlas_cluster_instance_size: M10

## The password that will be assigned to all database access users that are generated.
db_password: "SuperSecurePassword123!"
```

---

### 4. Run the playbook

Here are some example playbooks for managing your Atlas cluster.

#### Create the MongoDB Atlas Cluster
```
- hosts: localhost
  gather_facts: no
  tasks:
    - ansible.builtin.include_role:
        name: terraform
        tasks_from: create-atlas-cluster
```

This playbook will:
- Render the Terraform templates with your variables.
- Initialize and apply Terraform to create the MongoDB Atlas cluster.

#### Destroy the MongoDB Atlas Cluster
```
- hosts: localhost
  gather_facts: no
  tasks:
    - ansible.builtin.include_role:
        name: terraform
        tasks_from: destroy-atlas-cluster
```

#### Remove the local terraform artifacts
```
- hosts: localhost
  gather_facts: no
  tasks:
    - ansible.builtin.include_role:
        name: terraform
        tasks_from: remove-terraform-artifacts
```



---

## 📂 Repository structure
```
mongodb-atlas-ansible-role/
├── defaults/
├── handlers/
├── meta/
├── tasks/
│   ├── create-atlas-cluster.yml
│   ├── destroy-atlas-cluster.yml
│   ├── main.yml
│   └── remove-terraform-artifacts.yml
├── tests/
├── vars/
│   └── main.yml
├── README.md
└── requirements.yml
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
