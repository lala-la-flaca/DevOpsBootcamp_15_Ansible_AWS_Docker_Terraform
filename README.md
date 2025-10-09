# Module 15 – Configuration Management with Ansible
This exercise is part of Module 15 from the TWN DevOps Bootcamp. In Module 15, we focus on automating server setup and application deployment using Ansible. You learn how to configure servers, deploy Node.js and Nexus, integrate with Terraform and Jenkins, manage Docker containers, and organize playbooks with roles. Each demo builds practical automation skills for real-world DevOps environments.

<p align="left">
  <a href="#demo3">⚙️ Demo 3 - Ansible & Docker</a><br>
  <a href="#demo4">🚀 Demo 4 - Ansible integration in Terraform</a><br>
  <a href="#faq">❓ FAQ</a><br>
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
  Terraform files are available at: 🔗[demo/ansible-terraform](https://gitlab.com/devopsbootcamp4095512/devopsbootcamp_12_terraform_aws/-/tree/demo/ansible-terraform-2?ref_type=heads)
  
# 🏗 Project Architecture

# ⚙️ Project Configuration
## Terraform to deploy infrastructure
1. Use the Terraform file from the Terraform module 1 and remove the bootstrap section.
2. Initialize Terraform
   ```bash
   terraform init
   ```
3. Deploy AWS infrastructure using Terraform
   ```bash
   terraform apply --auto-approve
   ```
4. Check the Amazon console and verify that EC2s are running.
   
## Ansible to configure EC2
1. Copy the IP address from the EC2 server
2. Switch to the Ansible project.
3. Create the hosts file and add the IP address of the EC2.
4. Create the first play to install Docker.
5. Create a second play to install Docker-Compose.
   <details><summary><strong>Architecture of the Machine</strong></summary>
     uname: a command-line utility that prints basic information about the OS and hardware. This command runs as a shell command and passes the output to URL and obtain the latest linux version of the docker compose<br>
     uname -s: Linux <br>
     uname -m: x86_64 <br>
     The commands are encapsulated in the URL, the output of the uname-m is saved in the remote_arch variable, and passed to the URL.
   </details>
 6. Create a third play to start Docker.
 7. Create a fourth play to add the EC2-user to the Docker group.
    <details><summary><strong>Reset connection</strong></summary>
    After adding the user to the group, we must reset the connection so the changes are taken into account.
  </details>
 
 9. Create a fifth play to start Docker containers
[Community.Docker.Docker_ image module](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html)
10. Run the Ansible playbook.
    ```bash
    ansible-playbook 
    ```
---
<a id="demo4"></a>
# 📦Demo 4 – Ansible Integration in Terraform
# 📌 Objective
  Integrate Ansible into Terraform so Terraform automatically triggers Ansible playbooks after provisioning
  
# 🎯 Features
  ✅ End-to-end automation of infrastructure + configuration.<br>
  🐳 Terraform executes Ansible after creating servers.<br>
  ☁️ Consistent server setup on every provision.<br>
    
# ⚙️ Project Configuration
1. Using Terraform infrastructure from the Terraform module.
2. Add null_resource with provisioner "local-exec" to run Ansible.
3. Modify the previous playbook from module 3.


