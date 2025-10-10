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
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/1%20create%20infrastructur%20eusing%20terraform.PNG" width=800 />
  
3. Deploy AWS infrastructure using Terraform
   
   ```bash
   terraform plan
   terraform apply --auto-approve
   ```
   
4. Check the Amazon console and verify that EC2s are running.
   
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/2%20server%20ec2%20up.PNG" width=800/>

   
## Ansible to configure EC2
1. Copy the IP address from the EC2 instance.
   
2. Switch to the Ansible project directory.
   
3. Create a hosts file and add the EC2 IP address.
   
   ```bash
    [AWS_EC2_Docker_Server]
    3.89.217.238 ansible_ssh_private_key_file=~/.ssh/id_rsa ansible_user=ec2-user
   ```
   
   <img src="" width=800/>
   
4. Create the first play to install Docker.
   
   ```bash
    ---
    - name: Install Docker
      hosts: AWS_EC2_Docker_Server
      become: yes
      become_user: root
      tasks:
        - name: Installing Docker
          yum: 
            name: docker
            update_cache: yes
            state: present
    
        - name: Starting Docker daemon
          systemd:
            name: docker
            state: started
   ```
   
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/play1.PNG" width=800/>
   
5. Create a second play to install Docker-Compose.
   
   <details><summary><strong>Dynamic URL: uname </strong></summary>
     uname: a command-line utility that prints basic information about the OS and hardware. This command runs as a shell command and passes the output to a URL and obtain the latest Linux version of the Docker Compose <br>
     uname -s: Prints the name of the OS (e.g. Linux)<br>
     uname -m: Prints the architecture of the system. (e.g. x86_64) <br>
     These commands are used to dynamically build the URL that retrieves the latest Linux version of Docker Compose.
     The output of uname -m is stored in the remote_arch variable and passed to the URL.
   </details>
   
   ```bash
     - name: Installing Docker-Compose
      hosts: AWS_EC2_Docker_Server
      tasks:
        - name: Creating Docker-Compose directory
          file:
              path: ~/.docker/cli-plugins
              state: directory
        - name: Getting the architecture of the remote machine
          shell: uname -m
          register: remote_arch
        
        - debug: msg={{remote_arch.stdout}}
    
        - name: Installing Docker Compose
          get_url:
            url: "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-{{ remote_arch.stdout }}"
            dest: ~/.docker/cli-plugins/docker-compose
            mode: +x
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/play%202.png" width=800/>
    
 6. Create a fourth play to add the EC2-user to the Docker group.

    <details><summary><strong>Reset connection</strong></summary>
      After adding the user to the Docker group, reset the connection so the changes take effect.
    </details>
    
    ```bash
        #Allows EC2-user to execute Docker commands without sudo
        - name: Add ec2-user to Docker group
          hosts: AWS_EC2_Docker_Server
          become: yes
          tasks:
          - name: Adding ec2-user to docker group
            user: 
              name: ec2-user
              group: docker
              append: yes
         
          #In Ansible, we can reset the connection using the meta module as follows:
          - name: Resetting Remote Connection
            meta: reset_connection 
    ```
    
    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/play3.png" width=800/>
 
 7. Create a fifth play to start Docker containers using the module: [Community.Docker.Docker_ image module](https://docs.ansible.com/ansible/latest/collections/community/docker/docker_image_module.html)

     ```bash
       - name: Start Docker containers
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
     <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/play4.png" width=800/>
     
8. Run the Ansible playbook.
    ```bash
      ansible-playbook deploy-docker-ec2-user.yaml
    ```

9. Docker configured in EC2s
    
    <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/docker%20compose%20up%20and%20running%20in%20ec2.png" width=800/>
    

## Making the Project Reusable
1. Add a play to create a new user, instead of using the ec2-user.
   
   ```bash
     - name: Create new Linux user
    hosts: AWS_EC2_Docker_Server
    become: yes
    vars_files:
      project-vars.yaml
    tasks:
    - name: Creating new Linux user
    # Admin group in AWS is named adm.
      user: 
        name: "{{user_name}}"
        groups: "{{user_groups}}"
   ```
  
2. Modify the Docker Compose play to use the new user.
   
   ```bash
       - name: Installing Docker-Compose
        hosts: AWS_EC2_Docker_Server
        vars_files:
          project-vars.yaml
        become: yes
        become_user: "{{user_name}}"
        tasks:
          - name: Creating Docker Compose directory
            file:
                path: ~/.docker/cli-plugins
                state: directory
          - name: Getting the architecture of the remote machine
            shell: uname -m
            register: remote_arch
          
          - debug: msg={{remote_arch.stdout}}
      
          - name: Installing Docker Compose
            get_url:
              url: "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-{{ remote_arch.stdout }}"
              dest: ~/.docker/cli-plugins/docker-compose
              mode: +x
   ```
  
3. Modify the Start Docker containers play to start with the new user:
   
   ```bash
       - name: Start Docker containers
        hosts: AWS_EC2_Docker_Server
        vars_files:
            project-vars.yaml
        become: yes
        become_user: "{{user_name}}"
        tasks:
        - name: Copying Docker-compose yaml
          copy: 
            src: /home/lala/DevOpsBootCamp/ansible/demo3/docker-compose-java-mysql.yaml
            dest: /home/{{user_name}}/docker-compose.yaml
      
        - name: Logging to DockerHub registry
          #Default is DockerHub
          docker_login:
            username: your_username
            password: "{{password_docker_hub}}"
      
        - name: Starting Docker Compose
          community.docker.docker_compose_v2:
            project_src: /home/{{user_name}}
            #Equivalent to docker compose up
            #State absent: docker compose down
            state: present
   ```
---

<a id="demo4"></a>

# 📦Demo 4 – Ansible Integration in Terraform

# 📌 Objective
  Integrate Ansible into Terraform so Terraform automatically triggers Ansible playbooks after provisioning. Using the previous ansible configuration.
  
# 🎯 Features
  ✅ End-to-end automation of infrastructure + configuration.<br>
  🐳 Terraform executes Ansible after creating servers.<br>
  ☁️ Consistent server setup on every provision.<br>
    
# ⚙️ Project Configuration
1. Use Terraform infrastructure from the previous demo.
   [Terraform Files](https://gitlab.com/devopsbootcamp4095512/devopsbootcamp_12_terraform_aws/-/tree/demo/ansible-terraform?ref_type=heads)
  
2. Add a provisioner "local-exec" to run Ansible.
   
   <details><summary><strong> Terraform Provisioners</strong></summary>
     * Invokes a local executable after a resource is created <br>
     * The local-exec is applied to the host running Terraform and not the remote server.
     * In this case, we need the local server to execute the Ansible command.
   </details>
   
3. Obtain the IP address of the EC2 dynamically and pass it to Ansible, using the inventory flag.
   
   <details><summary><strong> IP address dynamically to Ansible </strong></summary>
     This is achieved using the --inventory flag when running the Ansible command. In this case, when running  ansible-playbook --inventory, we pass the EC2 instance IP address.
   </details>
   
4. Obtain the private key location and user to pass them to Ansible, using private key and user flags.
   
   ```bash
        provisioner "local-exec"{
        working_dir = "/home/lala/DevOpsBootCamp/ansible/demo3"
        command = "ansible-playbook --inventory ${aws_instance.myapp-ec2.public_ip}, --private-key ${var.ssh_key_private} --user ${var.username_ec2} deploy-docker-generic-terraform.yaml"
      }
   ```
5. Use null_resource to separate the provisioner from the AWS instance resource.
   
   <details><summary><strong> null_resource </strong></summary>
     null_resource to have provisioners in a different task <br>
     triggers --> Decides when to trigger the null_resource. In this case, the null_resource is executed when triggers finds changes in the EC2 IP address. <br>
   </details>
   
   ```bash
      resource "null_resource" "configure_server"{
      
        triggers = {
          trigger = aws_instance.myapp-ec2.public_ip
        }
      
        provisioner "local-exec"{
          working_dir = "/home/lala/DevOpsBootCamp/ansible/demo3"
          command = "ansible-playbook --inventory ${aws_instance.myapp-ec2.public_ip}, --private-key ${var.ssh_key_private} --user ${var.username_ec2} deploy-docker-generic-terraform.yaml"
        }
      }
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/null%20resource.PNG" width=800/>
   
6. Switch to Ansible configuration.
   
7. Modify the host from the docker_server group to ALL.
   
8. Ensure that the EC2 instance is accessible.
   ```bash
    #As we are executing Ansible from Terraform, we must ensure that the SSH connection is available before executing any command.
    - name: Wait for SSH connection
      hosts: all
      gather_facts: False
      tasks:
        - name: Ensuring SSH port is open
          wait_for:
            port: 22
            delay: 10
            timeout: 100
            search_regex: OpenSSH
            host: '{{(ansible_ssh_host|default(ansible_host))|default(inventory_hostname) }}'
          vars:
            ansible_connection: local
            ansible_python_interpreter: /usr/bin/python3
   ```
   <img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/ssh%20demo4.PNG" width=800/>
   
9. Switch to Terraform and apply the infrastructure.
   ```bash
     terraform init
     terraform plan
     terraform apply --auto-approve
   ```
   < img src="https://github.com/lala-la-flaca/DevOpsBootcamp_15_Ansible_AWS_Docker_Terraform/blob/main/Img/running%20aws%20ec2%20server%20form%20terraform%20using%20ansible%20provisioners.PNG" width=800/>
