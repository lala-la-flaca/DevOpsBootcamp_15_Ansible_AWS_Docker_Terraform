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
1. Copy the IP address from the EC2 instance.
2. Switch to the Ansible project directory.
3. Create a hosts file and add the EC2 IP address.
   ```bash
    [AWS_EC2_Docker_Server]
    3.89.217.238 ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=ec2-user
   ```
5. Create the first play to install Docker.
   ```bash
    ---
    - name: Install Docker
      hosts: AWS_EC2_Docker_Server
      become: yes
      become_user: root
      tasks:
        - name: Installing docker
          yum: 
            name: docker
            update_cache: yes
            state: present
    
        - name: Starting docker daemon
          systemd:
            name: docker
            state: started
   ```
7. Create a second play to install Docker-Compose.
   <details><summary><strong>Architecture of the Machine</strong></summary>
     uname: a command-line utility that prints basic information about the OS and hardware. This command runs as a shell command and passes the output to URL and obtain the latest linux version of the docker compose<br>
     uname -s: Prints the name of OS Ex. Linux<br>
     uname -m: Prints the architecture of the system, Ex. x86_64 <br>
     These commands are used to dynamically build the URL that retrieves the latest Linux version of Docker Compose.
     The output of uname -m is stored in the remote_arch variable and passed to the URL.
   </details>
   ```bash
     - name: Installing Docker-Compose
      hosts: AWS_EC2_Docker_Server
      tasks:
        - name: Creating docker-compose directory
          file:
              path: ~/.docker/cli-plugins
              state: directory
        - name: Getting architecture of remote machine
          shell: uname -m
          register: remote_arch
        
        - debug: msg={{remote_arch.stdout}}
    
        - name: Installing docker-compose
          get_url:
            url: "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-{{ remote_arch.stdout }}"
            dest: ~/.docker/cli-plugins/docker-compose
            mode: +x
   ```
    
 10. Create a fourth play to add the EC2-user to the Docker group.
    <details><summary><strong>Reset connection</strong></summary>
      After adding the user to the Docker group, reset the connection so the changes take effect.
    </details>
    ```bash
        #Allows EC2-user to execute docker comands without sudo
        - name: Add ec2-user to Docker group
          hosts: AWS_EC2_Docker_Server
          become: yes
          tasks:
          - name: Adding ec2-user to docker group
            user: 
              name: ec2-user
              group: docker
              append: yes
          # To consider the last change (ec2 user to the group) the connection must be reset to take effect.
          #In Ansible we can reset the connection the the remote machine using the meta module as follows:
          - name: Resetting Remote Connection
            meta: reset_connection 
    ```
 
 12. Create a fifth play to start Docker containers using the module: [Community.Docker.Docker_ image module](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html)
     ```bash
       - name: Start docker containers
        hosts: AWS_EC2_Docker_Server
        vars_files:
            project-vars.yaml
        tasks:
        - name: Copying Docker-compose yaml
          copy: 
            src: /home/lala/DevOpsBootCamp/ansible/demo3/docker-compose-java-mysql.yaml
            dest: /home/ec2-user/docker-compose.yaml
      
        - name: Logging to DockerHub registry
          #Default is DockerHub
          docker_login:
            username: lala.la.flaca11@gmail.com
            password: "{{password_docker_hub}}"
      
        - name: Starting Docker Compose
          community.docker.docker_compose_v2:
            project_src: /home/ec2-user
            #Equivalent to docker compose up
            #State absent: docker compose down
            state: present
     ```
14. Run the Ansible playbook.
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


