## Automation with Ansible: Implementing a Bastion Host and CI/CD Pipeline for Server Management
![Image](https://github.com/user-attachments/assets/12f032d8-ce26-4eb7-a353-0c93c1cce7f5)

### Introduction
This project focuses on implementing a secure and automated infrastructure management solution using Ansible as a Jump Server (Bastion Host). A Bastion Host acts as an intermediary server that provides controlled access to private network resources, improving security by preventing direct access to internal servers. By deploying Ansible on a Jenkins-Ansible EC2 instance, we automate server configurations and management tasks using playbooks. Additionally, GitHub and Jenkins are integrated to create a CI/CD pipeline, ensuring seamless deployment of infrastructure changes.
This project enhances security, automation, and efficiency, making it an essential practice for managing cloud infrastructure in a DevOps environment.

### Project Objectives
- To set up a Secure Bastion Host using an Ansible Jump Server for controlled access to private servers.
- To automate Server Configurations with Ansible playbooks for web, database, and load balancer servers.
- To integrate CI/CD Pipeline by linking Jenkins and GitHub for automated deployment of Ansible configurations.
- To organize Ansible Inventory to manage different environments like Development, Staging, user application testing (UAT), and Production.
- To test and Validate Automation by running Ansible playbooks to ensure proper server configuration and updates.

### Project Steps
1. Set Up Jenkins-Ansible Server
2. Create and Configure GitHub Repository
3. Integrate Jenkins with GitHub
4. Set Up Ansible Inventory
5. Develop Ansible Playbooks

### Project Implementation

### Step 1: Set Up Jenkins-Ansible Server
This step involves setting up a server that will act as both a Jenkins and Ansible control node. Jenkins is an automation server for CI/CD, while an Ansible Control Node is a machine that runs Ansible to automate remote system management.
- Spin up an Ubuntu Server. Go to AWS Console → EC2 → Launch Instance
- Note the below when launching the instance:
  - Select t2.medium
  - Storage: 30GB (recommended)
  - Configure Security Group:
    - SSH: Port 22 (Your IP only)
    - HTTP: Port 80 (Open to all, required for Jenkins)
    - HTTPS: Port 443 (Open to all)
    - Jenkins: Port 8080 (Your IP only)
      ![Image](https://github.com/user-attachments/assets/1c9c1d0f-4cda-4e2a-b6a0-343332adc377)
      
- Assign an Elastic IP (Avoid changing GitHub Webhook IP after restart)
  ![Image](https://github.com/user-attachments/assets/54a2d14b-5d90-4440-9d3c-0262590ed93f)
- Connect to your Jenkins-Ansible Server via VS Code. Open VS Code
- Go to Extensions (Ctrl + Shift + X), search for "Remote - SSH" the click Install
  ![Image](https://github.com/user-attachments/assets/486ba005-9fe1-4d46-8b0a-0af235b13c9e)
- Open VS Code and press Ctrl + Shift + P (Command Palette). Type "Remote-SSH: Open SSH Configuration File" and select it. Choose the config file to edit (~/.ssh/config on Linux/macOS or C:\Users\YourUser\.ssh\config on Windows).
- Paste the configuration
   ``sh
Host jenkins-ansible
 HostName 35.153.34.243
  User ubuntu
  IdentityFile C:/Users/SURFACE PRO 7PLUS/Downloads/DVops.pem
  ``
  ![Image](https://github.com/user-attachments/assets/67219a9d-7080-4c40-8b70-ae79c6dfb660)
  When running yours, do not forget to use your own host, host name, user and identity file.
  
- Press Ctrl + Shift + P, search "Remote-SSH: Connect to Host", and select it. Choose "jenkins-ansible" (or whatever name you used in the SSH config). VS Code will open a new SSH session into your server.
  ![Image](https://github.com/user-attachments/assets/e470cda3-4113-43d0-87af-1c80c794acc9)
  
- Connected successfully
![Image](https://github.com/user-attachments/assets/37b93115-bf28-4565-91b1-77b05dfdd324)



- Before installing Jenkins and Ansible, Update the system:
  ```sh
  sudo apt update -y
  ```
- Install Java (Required for Jenkins):
  ```sh
  sudo apt install openjdk-17-jdk -y
  ```
- Check Java version to confirm it was installed properly. Run;
  ```sh
  java -version
  ```
  ![Image](https://github.com/user-attachments/assets/5a6b1739-5095-43f1-bb41-11d08683b0be)
  
- To install Jenkins, we need to add the Jenkins repository key and Jenkins repository. Run;
  ```sh
  curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
  echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
  sudo apt update
  sudo apt install jenkins -y
  ```
  - Update the package list and install Jenkins
  ```sh
  sudo apt update
  sudo apt install jenkins -y
  ```
- Start and enable Jenkins:
  ```sh
  sudo systemctl start jenkins
  sudo systemctl enable jenkins
  ```
- Start and enable Jenkins:
  ```sh
  sudo systemctl status jenkins
  ```
  ![Image](https://github.com/user-attachments/assets/fb3386b8-7db5-4bb4-8a56-002ae307c986)
  
- Install Ansible:
  ```sh
  sudo apt install ansible -y
  ```
  - To confirm that ansible is installed, run;
  ```sh
  ansible --version
  ```
  ![Image](https://github.com/user-attachments/assets/a5fdc93b-512c-4904-a328-a9261472c938)

    - Finally, we need to configure our jenkins via browser. Paste your public IP:8080 in your browser in this format
  ```sh
  http://<EC2-Public-IP>:8080
  ```
  ![Image](https://github.com/user-attachments/assets/d40c9d4f-cce1-4e9c-adf9-81cd76ac4232)
  
- Run the command to get the Jenkins admin password. Once you get the password, copy and paste it on your browser where you are asked Administrator Password
  ```sh
  sudo cat /var/lib/jenkins/secrets/initialAdminPassword
  ```

  - Create your first admin user and you are logged into Jenkins
    ![Image](https://github.com/user-attachments/assets/cd261fe8-0079-4b55-a884-939a89d9980c)
    ![Image](https://github.com/user-attachments/assets/7dea8899-a531-4d68-9a60-2cbd042ad63c)
  

### Step 2: Create and Configure GitHub Repository
In this step, we'll set up a GitHub repository to store Ansible configurations and playbooks. This will allow us to track changes, collaborate, and integrate with Jenkins for automation.

- Create a GitHub repository `ansible-config-mgt`
  ![Image](https://github.com/user-attachments/assets/ff2c5c4e-bd69-4e45-9f93-b98c94913aab)
- Configure Git user details:
  ```sh
  git config --global user.name "Your Name"
  git config --global user.email "your-email@example.com"
  ```
- After running these commands, verify the configuration. Run;
```sh
git config –list
```
![Image](https://github.com/user-attachments/assets/4462a58d-5254-4b71-a7d9-a694811c92df)

- Clone the Repository to Your Local Machine. On your Jenkins-Ansible Server, navigate to the workspace directory. Run;
  ```sh
  git clone https://github.com/Ogbunugafor-Philip/ansible-config-mgt.git
  ```
  ![Image](https://github.com/user-attachments/assets/c7abca39-f079-439f-a45c-c6c0dfa7c2fb)
  
- Change directory to your ansible-config-mgt directory. Run;
  ```sh
  cd ansible-config-mgt
  ```
- Prepare Directory Structure. Inside the ansible-config-mgt repository, we would create two folders’ playbooks and inventory. Ansible works by running playbooks on target servers, which are listed in an inventory file. These two directories help us organize our Ansible automation setup. Run;
  ```sh
  mkdir playbooks inventory
  ```
- Verify That the Folders Were Created. Run the command to check the directory structure;
```sh
ls -R
```
- After setting up the repository structure, commit and push your changes to your github repository. Run; 
  ```sh
  git add .
  git commit -m "Initial commit: Added playbooks and inventory directories"
  git push origin main
  ```

### Step 3: Integrate Jenkins with GitHub
In this step, we will connect Jenkins to our GitHub repository so that Jenkins can automatically trigger builds when changes are pushed to the repository.
- We would first Install Git Plugin in Jenkins. To do that;
  Log in to your Jenkins web interface.
  Click "Manage Jenkins" from the left sidebar
  ![Image](https://github.com/user-attachments/assets/1304e854-4f40-4029-abb5-9ab915912272)
  Go to plugins, select available plugins by the left and in the search bar, type "Git 
  Plugin".
  Select "Git Plugin" and click "Install without restart".
  Wait for the installation to complete.

- Let us create a New Jenkins Job for GitHub Integration. In Jenkins, go to the Dashboard and click New Item.
  ![Image](https://github.com/user-attachments/assets/d9f26b8b-1488-415f-9998-d2eef68dcd04)

- Enter the name Ansible-Jump-Server then select free style project. Scroll down and click ok
  ![Image](https://github.com/user-attachments/assets/f116875a-9a62-422e-9389-7a7e5c186a42)

- Under the source code management, select git
  ![Image](https://github.com/user-attachments/assets/ef042b97-96bf-46aa-a46f-94d8d671fc2e)

- In the source code management UI, under the repository URL, paste your GitHub repository URL
  ![Image](https://github.com/user-attachments/assets/cf207b81-1ac8-44ee-a5f5-e81f1993ad69)

- Under branches, add your repo branch. My code resides in the main branch so will select 
  main then click save
  ![Image](https://github.com/user-attachments/assets/2324834b-009b-420a-a026-4f69242621d2)

- Click configure by the left-hand side. For your build trigger, select “GitHub hook trigger for GITSCM polling”. Scroll down and click ok.
  ![Image](https://github.com/user-attachments/assets/9418d469-60b1-44c4-a5de-24220fa598af)

- Now, lets go to our GitHub repo and make the following changes. On the repo, click on settings
  ![Image](https://github.com/user-attachments/assets/d3acca3c-a2c9-431e-bd30-0545dd27bee0)

- Scroll down under code and automation; select webhooks. Then click add webhook.
  ![Image](https://github.com/user-attachments/assets/cb3daa04-a600-4f4d-b6da-a2f0290b2a2d)

- In the webhook UI, under the payload URL, paste in your jenkins server IP in the below format shown below, then click add webhook
```sh
http://35.153.34.243:8080/github-webhook/
```
![Image](https://github.com/user-attachments/assets/54f8c7f7-abd9-452a-bbe6-53f16db42cb9)

- To test your pipeline, come to jenkins project dashboard, then click play button to build. Under build, you would see your first project is successful.
  ![Image](https://github.com/user-attachments/assets/d98914a9-54ee-4b95-adca-2be5c1ddf6b2)

  

### Step 4: Set Up Ansible Inventory
An Ansible inventory file defines the hosts and groups of hosts upon which commands, modules, and tasks in a playbook operate. Since our intention is to execute Linux commands on remote hosts, and ensure that it is the intended configuration on a particular server that occurs. It is important to have a way to organize our hosts in such an Inventory.

- Create an inventory file:
  ```sh
  cd inventory
  touch host.yml
  ```
- We have 3 server (2 webservers and 1 database server) that we want to manage. When managing servers on same VPC, use Private IP but if servers are on different VPC, use Public IP.
Paste the below in your host.yml
```sh
all:
  children:
    webservers:
      hosts:
        webserver01:
          ansible_host: 172.31.27.45
          ansible_user: ubuntu
          ansible_ssh_private_key_file: /home/ubuntu/.ssh/DVops.pem
        webserver02:
          ansible_host: 172.31.23.121
          ansible_user: ubuntu
          ansible_ssh_private_key_file: /home/ubuntu/.ssh/DVops.pem
    dbservers:
      hosts:
        dbserver:
          ansible_host: 172.31.28.248
          ansible_user: ubuntu
          ansible_ssh_private_key_file: /home/ubuntu/.ssh/DVops.pem
```
![Image](https://github.com/user-attachments/assets/2ad5e0e9-7457-44b7-a1b6-70cd0eb9cd2a)

- Next, we would need to test to see if our host.yml works properly by trying to ping all the servers. Run;
  ```sh
  ansible -i inventory/host.yml all -m ping
  ```
  ![Image](https://github.com/user-attachments/assets/979a8832-3ce0-4658-bb9e-ebacac02b6ac)

### Step 5: Develop Ansible Playbooks
Next, we would develop an Ansible Playbook. An Ansible Playbook is a YAML file that contains a set of tasks that Ansible executes on remote servers. Instead of running individual commands, you define the desired system state, and Ansible ensures that the servers match that state.
Basically, an ansible playbook has this structure;
A basic Ansible playbook has:
   - Hosts: Which servers to run tasks on
   - Become: Whether to use sudo for elevated permissions
   - Tasks: List of operations to perform
   - Conditionals: Optional conditions to control when tasks run

- Create a playbook file:
  ```sh
  cd playbooks
  touch setup.yml
  ```
- Define tasks in `setup.yml`
  Paste the below on your setup.yml file
```sh
---
- name: Setup Web and DB Servers
  hosts: all
  become: yes  
  tasks:
  
    - name: Update package lists
      apt:
        update_cache: yes

    - name: Install Nginx on Web Servers
      apt:
        name: nginx
        state: present
      when: "'webservers' in group_names"

    - name: Install MySQL on DB Server
      apt:
        name: mysql-server
        state: present
      when: "'dbservers' in group_names"
```
![Image](https://github.com/user-attachments/assets/13fd09d8-3604-4aa2-b7b9-79b94ae4c4cd)

What This Playbook Does:
   - Updates the package list on all servers
   - Installs nginx on servers in the webservers group
   - Installs mysql-server on the dbserver


- Run the playbook:
  ```sh
  ansible-playbook -i ~/ansible-config-mgt/inventory/host.yml ~/ansible-config-mgt/playbooks/setup.yml
  ```
  ![Image](https://github.com/user-attachments/assets/a33324be-5a18-4443-9ca1-9a01215ef894)
  
- Our setup.yml playbook ran perfectly well. Now we would ssh into 1 web server and the db server to confirm our installations are actually done.
ssh into one of the web servers. Run; 
```sh
ssh -i ~/.ssh/DVops.pem ubuntu@172.31.27.45
```
- Confirm nginx is installed. Run;
```sh
nginx -v
```
![Image](https://github.com/user-attachments/assets/03f8e9cd-0790-441e-bc20-457696b14791)

- Next, ssh into the db server and confirm mysql is installed. Run;
```sh
ssh -i ~/.ssh/DVops.pem ubuntu@172.31.28.248
```
- Next, confirm mysql version
```sh
mysql –version
```
![Image](https://github.com/user-attachments/assets/03cd9c2f-7ce9-4654-96f1-658cb8b46215)

### Conclusion
This project successfully implemented secure infrastructure automation using Ansible as a Jump Server (Bastion Host), combined with Jenkins and GitHub for CI/CD-driven deployment.
By setting up an Ansible-based Bastion Host, we ensured controlled and secure access to private servers, preventing unauthorized access while centralizing automation.
Through Jenkins integration, Ansible configurations are automatically deployed whenever updates are pushed to GitHub, ensuring a continuous and seamless deployment pipeline.
   - The Ansible Playbooks effectively automated server provisioning, ensuring that:
      Web servers are properly configured with Nginx 
   - Database servers have MySQL installed and running
   - All servers remain up-to-date and secured
The implementation of GitHub Webhooks further streamlined the process, allowing changes in the repository to trigger automatic deployments without manual intervention.
This project enhances cloud security, operational efficiency, and infrastructure automation, making it a valuable solution for managing cloud environments in a DevOps workflow.


This project enhances cloud security, operational efficiency, and infrastructure automation, making it a valuable solution for managing cloud environments in a DevOps workflow.

