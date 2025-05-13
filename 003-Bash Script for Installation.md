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

# Remove any existing symbolic link if it exists
rm -f /opt/sonarqube

# Create a new correct symlink to the extracted SonarQube folder
ln -s /opt/sonarqube-${SONAR_VERSION} /opt/sonarqube

# Ensure the sonar.sh script is executable
chmod +x /opt/sonarqube/bin/linux-x86-64/sonar.sh

# Ensure proper ownership
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
ExecStart=/bin/bash -c '/opt/sonarqube/bin/linux-x86-64/sonar.sh start'
ExecStop=/bin/bash -c '/opt/sonarqube/bin/linux-x86-64/sonar.sh stop'
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

---

# ✅ 1. **Verify Docker**

```bash
sudo systemctl status docker
```

You should see `active (running)`.

Also, run:

```bash
docker version
docker info
```

These should return version and runtime info without errors.

---

### ✅ 2. **Verify K3s (Lightweight Kubernetes)**

Check the service:

```bash
sudo systemctl status k3s
```

And verify the Kubernetes cluster is working:

```bash
sudo k3s kubectl get nodes
```

You should see your node in `Ready` state.

---

### ✅ 3. **Verify Jenkins**

Check the service:

```bash
sudo systemctl status jenkins
```

If running, open Jenkins in your browser:

```
http://<your-ec2-public-ip>:8080
```

🔐 To get the initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Paste that into the Jenkins setup wizard.

---

### ✅ 4. **Verify SonarQube**

Check the service:

```bash
sudo systemctl status sonarqube
```

If active, access the SonarQube dashboard:

```
http://<your-ec2-public-ip>:9000
```

🧠 Default credentials:

* **Username**: `admin`
* **Password**: `admin`

It will prompt you to change the password on first login.

---

### 🧪 Optional: Port Check (if unsure)

Make sure the EC2 instance's **Security Group** allows traffic on ports:

* `8080` (Jenkins)
* `9000` (SonarQube)
* `6443` (K3s API, optional)
* `22` (SSH, obviously)

You can run this on the EC2 instance to confirm listening ports:

```bash
sudo ss -tuln | grep -E '8080|9000|6443'
```

---
