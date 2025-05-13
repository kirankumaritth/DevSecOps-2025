# 📋 **2. PLAN: Ansible for Configuration Management**

In this step, we'll use **Ansible** to automate the installation and configuration of the required software on the newly provisioned EC2 instance. The key software includes:

* **Docker**: For containerizing applications.
* **K3s (Kubernetes)**: Lightweight container orchestration.
* **Jenkins**: To orchestrate the CI/CD pipeline.
* **SonarQube**: For static code analysis.
* **Trivy**: For scanning Docker images for vulnerabilities.

We’ll define the following:

1. **Ansible Inventory**: Defines the EC2 instance(s) to be configured.
2. **Playbooks**: Automates the installation and setup of Docker, K3s, Jenkins, SonarQube, and Trivy.
3. **Roles** (optional): To structure the setup for Docker, Kubernetes, etc., in separate reusable modules.

---

### 🔑 **Ansible Configuration**

#### 1. **Ansible Inventory File**

Create an inventory file to define the EC2 instance(s) you’ll be managing with Ansible. This file could be named `hosts.ini`:

```ini
[devsecops]
ec2_instance ansible_host=your_ec2_public_ip ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/your_ssh_key.pem
```

* Replace `your_ec2_public_ip` with the public IP of the EC2 instance.
* The `ansible_user` is typically `ubuntu` for Ubuntu EC2 instances.
* The `ansible_ssh_private_key_file` should point to your SSH private key to authenticate the connection.

#### 2. **Ansible Playbook**

Next, create an **Ansible playbook** to configure the instance. This playbook can be divided into multiple tasks, including Docker, K3s, Jenkins, etc. Below is a simplified version of the playbook:

```yaml
---
- name: DevSecOps Setup
  hosts: devsecops
  become: true
  tasks:

    - name: Update apt repository
      apt:
        update_cache: yes
        upgrade: dist

    - name: Install Docker
      apt:
        name: docker.io
        state: present

    - name: Install Kubernetes (K3s) using script
      shell: |
        curl -sfL https://get.k3s.io | sh -
      args:
        creates: /usr/local/bin/k3s

    - name: Enable K3s service
      service:
        name: k3s
        state: started
        enabled: yes

    - name: Install Jenkins
      shell: |
        curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | tee \
          /usr/share/keyrings/jenkins.asc
        sh -c "echo deb http://pkg.jenkins.io/debian/ / > /etc/apt/sources.list.d/jenkins.list"
        apt-get update
        apt-get install -y jenkins
      args:
        creates: /usr/share/keyrings/jenkins.asc

    - name: Start Jenkins service
      service:
        name: jenkins
        state: started
        enabled: yes

    - name: Install SonarQube
      shell: |
        sudo apt-get update
        sudo apt-get install -y openjdk-11-jdk
        sudo apt-get install -y unzip
        wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.7.1.59510.zip
        unzip sonarqube-9.7.1.59510.zip -d /opt
        sudo ln -s /opt/sonarqube-9.7.1.59510 /opt/sonarqube
        sudo useradd sonar
        sudo chown -R sonar:sonar /opt/sonarqube
      args:
        creates: /opt/sonarqube

    - name: Start SonarQube
      service:
        name: sonarqube
        state: started
        enabled: yes
```

#### 3. **Explanation of Tasks in the Playbook**

1. **Update apt repository**: Updates the package repository on the EC2 instance.
2. **Install Docker**: Installs Docker using the `docker.io` package.
3. **Install K3s**: Uses the official K3s install script to install a lightweight Kubernetes setup.
4. **Install Jenkins**: Installs Jenkins from the official Debian repository.
5. **Install SonarQube**: Installs Java, downloads SonarQube, and sets it up to run.
6. **Start Jenkins and SonarQube**: Ensures Jenkins and SonarQube services are up and running.

#### 4. **Running the Playbook**

To run the playbook, use the following command from your local machine or wherever you have Ansible installed:

```bash
ansible-playbook -i hosts.ini setup-devsecops.yml
```

This command will initiate the configuration of your EC2 instance by connecting over SSH, installing the necessary tools, and ensuring they’re running.

---

### 🛠️ **Optional Role-Based Setup** (Best Practice)

For a more modular and reusable setup, you can create **roles** for each software (Docker, K3s, Jenkins, etc.). Here’s how you can structure it:

```bash
ansible/
│
├── roles/
│   ├── docker/
│   ├── k3s/
│   ├── jenkins/
│   └── sonarqube/
│
└── setup-devsecops.yml
```

Each role (e.g., `docker/`, `k3s/`) would contain its own set of tasks, defaults, and handlers. You can then include the roles in the main playbook.

```yaml
---
- name: DevSecOps Setup
  hosts: devsecops
  become: true
  roles:
    - docker
    - k3s
    - jenkins
    - sonarqube
```

---
