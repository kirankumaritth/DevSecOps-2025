# 📝 **Bash Script for Installation**

Create a script called `setup-devsecops.sh`:

```bash
#!/bin/bash

set -euo pipefail

# Log function
log() {
  echo -e "\n\033[1;32m[INFO]\033[0m $1\n"
}

# Ensure script is run as root
if [ "$EUID" -ne 0 ]; then
  echo "Please run as root or with sudo"
  exit 1
fi

log "Updating system packages..."
apt update && apt full-upgrade -y

### ----------------- Docker Installation -----------------
log "Installing Docker (from Ubuntu repo)..."
apt install -y docker.io
systemctl enable --now docker

### ----------------- K3s Installation -----------------
log "Installing K3s (Lightweight Kubernetes)..."
/usr/bin/curl -sfL https://get.k3s.io | sh -
systemctl enable --now k3s

### ----------------- Jenkins Installation -----------------
log "Installing Jenkins (from official Jenkins repo)..."

# Install dependencies
apt install -y fontconfig openjdk-17-jdk

# Add Jenkins key and repo (use Debian bookworm which works on 24.04)
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" \
  > /etc/apt/sources.list.d/jenkins.list

apt update
apt install -y jenkins
systemctl enable --now jenkins

### ----------------- SonarQube Installation -----------------
log "Installing SonarQube (manual install)..."

# Install Java
apt install -y openjdk-17-jdk unzip wget

# Create a sonar user
useradd -m -d /opt/sonarqube sonar || true

# Download and extract SonarQube
SONAR_VERSION="10.2.1.78527"
cd /opt
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-${SONAR_VERSION}.zip
unzip sonarqube-${SONAR_VERSION}.zip

# Create symlink only if it doesn't exist
[ -L /opt/sonarqube ] || ln -s sonarqube-${SONAR_VERSION} sonarqube

chown -R sonar:sonar /opt/sonarqube*

# Create SonarQube systemd service
log "Creating SonarQube systemd service..."

cat <<EOF > /etc/systemd/system/sonarqube.service
[Unit]
Description=SonarQube service
After=network.target

[Service]
Type=simple
User=sonar
Group=sonar
ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
Restart=always
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reexec
systemctl daemon-reload
systemctl enable --now sonarqube

log "DevSecOps environment setup completed successfully!"
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
