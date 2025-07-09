
# MongoDB Atlas Ansible Role

This repository contains an Ansible playbook and role that dynamically generates Terraform configuration files using Jinja2 templates, and then applies them to provision and destroy MongoDB Atlas clusters.

It enables you to use Ansible variables to control your MongoDB Atlas infrastructure, combining the simplicity of Ansible with the power of Terraform for managing cloud resources.

---

## 🚀 Features

- Generate Terraform `main.tf`, `providers.tf`, and `vars.tf` dynamically from Ansible variables.
  - Uses Jinja2 templating for flexibility and reusability.
- Supports creating MongoDB Atlas clusters with selected ansible variables as described below.
  - Automate Terraform init, plan, and apply from Ansible.

---

## 📂 Role Repository Structure
```
mongodb-atlas-ansible-role/
├── defaults/
│   └── main.yml
├── handlers/
├── meta/
├── tasks/
│   ├── create-atlas-cluster.yml
│   ├── destroy-atlas-cluster.yml
│   ├── main.yml
│   └── remove-terraform-artifacts.yml
├── templates/
│   ├── main.tf.j2
│   ├── providers.tf.j2
│   └── vars.tf.j2
├── tests/
├── vars/
├── README.md
└── requirements.yml
```

---

## ✅ MongoDB Atlas Requirements

- **Existing Atlas project:**  
  - The MongoDB Atlas project must already exist. This automation does **not create the Atlas project**, only the cluster inside it. You must provide the `atlas_project_name` and `atlas_project_field` values as specified below.

- **Project-level API keys:**  
  - The API keys used must be created under the same Atlas project and have at least `Project Owner` permissions.
    - See instructions on how to create MongoDB Atlas API keys for your project [at this link](https://www.mongodb.com/docs/atlas/configure-api-access/#grant-programmatic-access-to-a-project).

___

## 🛠 Usage

### 1. Local Machine Requirements

#### Prerequisites 

You can use this Ansible role on nearly any UNIX-like machine with Python installed. This includes Red Hat, Debian, Ubuntu, and macOS.

Make sure you have the following installed:

- [Ansible >= 2.9](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [Terraform >= 1.0](https://developer.hashicorp.com/terraform/install)
- Python packages may be needed for Ansible collections. Check the collection pages on [Ansible Galaxy](https://galaxy.ansible.com/) to see which packages to install.

#### Install Ansible mongodb-atlas-ansible-role Role and associated Collections

##### Via the requirements.yml file
Be sure to require the following collections in your `requirements.yml` file in the same directory as your playbook:

```
# requirements.yml
roles:
  - name: mongodb-atlas-ansible-role
    src: git+https://github.com/michaelford85/mongodb-atlas-ansible-role.git
collections:
  - name: community.mongodb
    version: 1.7.10
  - name: community.general
    version: 11.0.0
  - name: ansible.posix
    version: 2.0.0
```

You can then run the following command:
`ansible-galaxy install -r requirements.yml`

##### Individual collection installation

Alternatively, you install them manually with the following commands:

```bash
ansible-galaxy install git+https://github.com/michaelford85/mongodb-atlas-ansible-role.git
ansible-galaxy collection install community.mongodb:1.7.10
ansible-galaxy collection install community.general:10.0.0
ansible-galaxy collection install ansible.posix:2.0.0
```

---

### 2. Define Role Variables
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

## The MongoDB Atlas API Public Key. 
## In practice, this should not be stored directly in a plaintext variable file. 
## Store it in a separate file encrypted with Ansible Vault and reference it as shown here.
vaulted_atlas_public_key: "abcd1234efgh5678ijkl"

## The MongoDB Atlas API Private Key.
## As with the public key, this should be stored securely in an encrypted file 
## (for example using Ansible Vault) and not committed to source control.
vaulted_atlas_private_key: "mnop9012qrst3456uvwx7890yzab1234"

## This is the the desired name of the cluster, and will also be incorporated into the
## database access usernames that will be generated.
atlas_cluster_name: test-cluster

## The desired instance type of replica set members in your cluster
## It can range from M10 to M80.
atlas_cluster_instance_size: M10

## The password that will be assigned to all database access users that are generated.
## In practice, this should not be stored directly in a plaintext variable file. 
## Store it in a separate file encrypted with Ansible Vault and reference it as shown here.
vaulted_db_password: "S3cur3P@ssw0rd!XyZ789"
```

---

### 3. Run the playbook

After installing the role and other prerequisites, author a playbook in order to create/destroy your cluster. Below you will find an example directory with files and playbooks to run.

#### 📂 Example Playbook Repository Structure
```
atlas-orchestrator/
├── credentials/
│   └── atlas_creds.yml
├── ansible.cfg
├── create_cluster.yml
├── destroy_cluster.yml
├── remove_tf_artifacts.yml
├── inventory
└── requirements.yml
```

##### atlas_creds.yml

This file will hold your Atlas API keys and database user password. Use [Ansible vault](https://docs.ansible.com/ansible/latest/vault_guide/index.html) to encrypt this file for safe keeping.

```
---
vaulted_atlas_public_key: "abcd1234efgh5678ijkl"
vaulted_atlas_private_key: "mnop9012qrst3456uvwx7890yzab1234"
vaulted_db_password: "S3cur3P@ssw0rd!XyZ789"
```

##### inventory

An inventory file that defines the hosts Ansible will manage. In this example, it simply points to `localhost` using a local connection, since all operations (templating, Terraform, API calls) are executed from your local machine.

```
localhost ansible_connection=local
```

##### requirements.yml

Specifies the Ansible collections required by this project, ensuring all necessary modules and plugins are available. Install these dependencies with `ansible-galaxy install -r requirements.yml`.
```
---
roles:
  - name: mongodb-atlas-ansible-role
    src: git+https://github.com/michaelford85/mongodb-atlas-ansible-role.git
collections:
  - name: community.mongodb
    version: 1.7.10
  - name: community.general
    version: 11.0.0
  - name: ansible.posix
    version: 2.0.0
```

##### ansible.cfg

Configuration file for Ansible. In this example, it specifies where to find your inventory file, sets the location of your roles and collections directories (so Ansible can discover them locally), and adjusts SSH connection settings and timeouts for more robust runs.

```
[defaults]
inventory      = inventory
forks          = 50
host_key_checking = False
roles_path = ./roles
collections_path = ./collections

[persistent_connection]
command_timeout = 200
connect_timeout = 200
```

##### create_cluster.yml
```
---
- hosts: localhost
  gather_facts: no
  vars:
    atlas_org_id: 1234567890abcdef12345678
    atlas_project_name: "Test Project"
    atlas_project_id: 1234567890abcdef12345679
    atlas_cluster_name: test-cluster
    atlas_cluster_instance_size: M10
  
  # An encrypted ansible variable file housing the variables:
  #  - vaulted_atlas_public_key
  #  - vaulted_atlas_private_key
  #  - vaulted_db_password
  vars_files:
    - credentials/atlas_creds.yml

  tasks:
    - ansible.builtin.include_role:
        name: mongodb-atlas-ansible-role
        tasks_from: create-atlas-cluster
```

##### destroy_cluster.yml
```
---
- hosts: localhost
  gather_facts: no
  vars:
    atlas_org_id: 1234567890abcdef12345678
    atlas_project_name: "Test Project"
    atlas_project_id: 1234567890abcdef12345679
    atlas_cluster_name: test-cluster
  
  # An encrypted ansible variable file housing the variables:
  #  - vaulted_atlas_public_key
  #  - vaulted_atlas_private_key
  vars_files:
    - credentials/atlas_creds.yml

  tasks:
    - ansible.builtin.include_role:
        name: mongodb-atlas-ansible-role
        tasks_from: destroy-atlas-cluster
```

##### remove_tf_artifacts.yml
```
---
- hosts: localhost
  gather_facts: no
  vars:
    atlas_cluster_name: test-cluster

  tasks:
    - ansible.builtin.include_role:
        name: mongodb-atlas-ansible-role
        tasks_from: remove-terraform-artifacts
```

#### Playbook Commands:

##### Create the cluster
```bash
ansible-playbook create_cluster.yml
```

If your `atlas_creds.yml` is encrypted with Ansible Vault, include the `--vault-id` flag:

```bash
ansible-playbook create_cluster.yml --vault-id @prompt
```

##### Destroy the cluster
```bash
ansible-playbook destroy_cluster.yml
```

If your `atlas_creds.yml` is encrypted:

```bash
ansible-playbook destroy_cluster.yml --vault-id @prompt
```

##### Remove local Terraform artifacts
```bash
ansible-playbook remove-tf-artifacts.yml
```
---

## 🚧 Future enhancements

- Add support for **conditionally creating database users**, allowing users to skip default user creation if not needed.
- Provide the ability to deploy **free tier clusters** (M0) or select higher-tier configurations based on use case.
- Expand cluster customization options, enabling users to fully define their MongoDB Atlas cluster via Ansible variables and Terraform, including but not limited to:
  - Choice of **cloud provider** (AWS, Azure, GCP).
  - Configuration of **number of electable nodes** and **number of read-only nodes** for advanced replication setups.
  - Enable or disable **cloud backup** and **continuous cloud backup** options.
  - Support for **sharded cluster deployments**, including setting number of shards and configuration servers.
  - Ability to set **cluster tags** for organizational or billing purposes.

The goal is to allow this role to act as a comprehensive interface for creating any MongoDB Atlas cluster configuration supported by the Terraform provider, fully managed through Ansible variables.

---

## 🤝 Contributing
PRs and issues are welcome! Please open an issue first to discuss changes or new features.

---

## 📬 Contact
Created by [Michael Ford](https://github.com/michaelford85) - feel free to reach out!

---