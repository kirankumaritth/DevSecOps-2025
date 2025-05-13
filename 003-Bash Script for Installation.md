# 📝 **Bash Script for Installation**

Create a script called `setup-devsecops.sh`:

```bash
#!/bin/bash

# Update and upgrade system
echo "Updating system..."
sudo apt-get update -y
sudo apt-get upgrade -y

# Install Docker
echo "Installing Docker..."
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker

# Install K3s (Lightweight Kubernetes)
echo "Installing K3s..."
curl -sfL https://get.k3s.io | sh -
sudo systemctl enable k3s
sudo systemctl start k3s

# Install Jenkins
echo "Installing Jenkins..."
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | tee /usr/share/keyrings/jenkins.asc
echo "deb http://pkg.jenkins.io/debian/ /" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt-get update -y
sudo apt-get install -y jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins

# Install SonarQube
echo "Installing SonarQube..."
sudo apt-get install -y openjdk-11-jdk unzip wget
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-9.7.1.59510.zip
sudo unzip sonarqube-9.7.1.59510.zip -d /opt
sudo ln -s /opt/sonarqube-9.7.1.59510 /opt/sonarqube
sudo useradd sonar
sudo chown -R sonar:sonar /opt/sonarqube
sudo systemctl enable sonarqube
sudo systemctl start sonarqube

# Install Trivy (For Docker image scanning)
echo "Installing Trivy..."
sudo apt-get install -y apt-transport-https
echo "deb https://aquasecurity.github.io/trivy-repo/deb main" | sudo tee /etc/apt/sources.list.d/trivy.list
curl -fsSL https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo tee /etc/apt/trusted.gpg.d/trivy.asc
sudo apt-get update -y
sudo apt-get install -y trivy

# Print the installation status
echo "Installation complete!"
echo "Docker version: $(docker --version)"
echo "K3s version: $(k3s --version)"
echo "Jenkins status: $(systemctl is-active jenkins)"
echo "SonarQube status: $(systemctl is-active sonarqube)"
echo "Trivy version: $(trivy --version)"
```

### 🛠️ **Explanation of the Script**

1. **System Update**: The script first updates the system using `apt-get update` and `apt-get upgrade` to ensure that all the repositories and packages are up-to-date.
2. **Docker**: Installs Docker via the official `docker.io` package and starts the service.
3. **K3s**: Installs K3s using the official install script and starts the service.
4. **Jenkins**: Adds the Jenkins repository, installs Jenkins, and starts the service.
5. **SonarQube**: Installs Java, downloads SonarQube, unzips it, and configures it to run as a service.
6. **Trivy**: Adds the Trivy repository, installs it, and ensures it's ready for scanning Docker images.

### ▶️ **Running the Script**

To run the script on your EC2 instance:

1. **Upload the script to your EC2 instance** (or create it directly on the instance).
2. **Give the script executable permissions**:

```bash
chmod +x setup-devsecops.sh
```

3. **Run the script**:

```bash
./setup-devsecops.sh
```

### ⚠️ **Considerations**

* Ensure your EC2 instance has proper internet access to download the packages.
* Depending on your setup, you may need to adjust security groups or firewall settings to access Jenkins, SonarQube, or other services from outside the EC2 instance.
* For production setups, consider adding additional configurations for each tool (e.g., setting up Jenkins jobs, SonarQube database, Trivy scanning options, etc.).

This bash script provides an easy and straightforward method to get your DevSecOps tools installed and running. Let me know if you'd like help modifying or expanding it!
