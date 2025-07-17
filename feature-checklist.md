
# 🚀 MongoDB Atlas Ansible Role Enhancement Roadmap

Follow this checklist to progressively enhance the role with new features.

---

## ✅ Enhancement checklist

- [x] **Free vs paid cluster toggle**
  - Add `atlas_cluster_instance_size` defaulted to `M10` in `defaults/main.yml`.
  - Allow users to specify `deploy_free_cluster: true` to deploy a free cluster.
    - Ignore all other options when `deploy_free_cluster: true`
  - Document example usage in README.

- [] **Cost Estimation**
  - For a paid cluster, add a dry run option to estimate the costs per month for an Atlas Cluster with the chosen options.

- [ ] **Option to skip creating default database users**
  - Add `create_default_db_user: true` to `defaults/main.yml`.
  - Use `count` or `for_each` in Terraform Jinja2 templates to optionally skip `mongodbatlas_database_user` resource.
  - Document how to disable in README.

- [ ] **Cloud provider and region options**
  - Expose `provider_name` (AWS, AZURE, GCP) and `region_name` as variables.
  - Pass to Terraform template to dynamically create cluster in selected cloud.
  - Document examples in README.

- [ ] **Cluster shape & resilience**
  - Add support for `number of electable nodes`, `read-only nodes`, `analytics nodes`.
  - Expose these as variables in `defaults/main.yml` and wire into Terraform template.
  - Add support for `num_shards` to enable sharding.

- [ ] **Cloud backup / continuous backup options**
  - Add variables like `provider_backup_enabled`, `continuous_backup_enabled`.
  - Integrate into Terraform template.
  - Validate with instance type where applicable.

- [ ] **Cluster tags**
  - Allow users to specify a `tags` map variable.
  - Use `for_each` in Terraform to apply cluster tags.

---

## 🚀 Tips for each step

- Create a feature branch for each enhancement.
- Update `defaults/main.yml` to introduce new variables with sensible defaults.
- Adjust your Jinja2 templates (`main.tf.j2`, etc.) to consume these variables.
- Write or update example playbooks under `tests/` to validate each new feature.
- Update the `README.md` after each enhancement with examples and usage instructions.

---

✅ **Done!**  
Check each item as you complete it to track progress.
