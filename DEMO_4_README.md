# 📦Demo 4 – Ansible Integration in Terraform
# 📌 Objective
  Integrate Ansible into Terraform so Terraform automatically triggers Ansible playbooks after provisioning
  
# 🎯 Features
  ✅ End-to-end automation of infrastructure + configuration.
  🐳 Terraform executes Ansible after creating servers
  ☁️ Consistent server setup on every provision
  
# ⚙️ Project Configuration
1. Using Terraform infrastructure from the Terraform module.
2. Add null_resource with provisioner "local-exec" to run Ansible.
3. Modify the previous playbook from module 3.
