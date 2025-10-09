# Module 15 – Configuration Management with Ansible
This exercise is part of Module 15 from the TWN DevOps Bootcamp. In Module 15, we focus on automating server setup and application deployment using Ansible. You learn how to configure servers, deploy Node.js and Nexus, integrate with Terraform and Jenkins, manage Docker containers, and organize playbooks with roles. Each demo builds practical automation skills for real-world DevOps environments.

<p align="center">
  <a href="#demo3">⚙️ Demo 3 - Ansible & Docker</a>  ·
  <a href="#demo4">🚀 Demo 4 - Ansible integration in Terraform</a>  ·
  <a href="#faq">❓ FAQ</a>  ·
</p>

---
<a id="demo3"></a>
# 📦Demo 3 – Ansible & Docker
# 📌 Objective
Use Terraform and Ansible to deploy Docker and Docker Compose on AWS EC2 instances.

# 🚀 Technologies Used
* Ansible: Configuration management tool for automation.
* Terraform: Provisions AWS infrastructure.
* AWS: Cloud provider.
* Linux: OS.

# 🎯 Features
✅ Provisions EC2 instances using Terraform.<br>
🐳 Installs Docker & Docker Compose via Ansible.<br>
🧩 Deploy nginx application from a compose file.<br>

# Prerequisites
* AWS account with valid keys.
* Terraform demo to deploy infrastructure.
  
# 🏗 Project Architecture

# ⚙️ Project Configuration

---
<a id="demo4"></a>
# 📦Demo 4 – Ansible Integration in Terraform
# 📌 Objective
  Integrate Ansible into Terraform so Terraform automatically triggers Ansible playbooks after provisioning
  
# 🎯 Features
✅ End-to-end automation of infrastructure + configuration.  
🐳 Terraform executes Ansible after creating servers.  
☁️ Consistent server setup on every provision.  
  
# ⚙️ Project Configuration
1. Using Terraform infrastructure from the Terraform module.
2. Add null_resource with provisioner "local-exec" to run Ansible.
3. Modify the previous playbook from module 3.


