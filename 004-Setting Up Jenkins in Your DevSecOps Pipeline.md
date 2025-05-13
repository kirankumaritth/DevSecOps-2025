# 🚀 **Setting Up Jenkins in Your DevSecOps Pipeline**

We'll dive deeper into setting up **Jenkins** as part of your DevSecOps pipeline. These steps will cover:

1. **Installing Jenkins on Your EC2 Instance**.
2. **Configuring Jenkins to Integrate with GitHub** for CI/CD automation.
3. **Setting Up Jenkins Jobs** for building, testing, and deploying applications.

---

### 🛠️ **1. Installing Jenkins on Your EC2 Instance**

Since we previously installed Jenkins through the Ansible playbook, let’s take a closer look at securing and configuring Jenkins for your pipeline.

#### Steps:

1. **Start Jenkins Service**:
   Ensure Jenkins is running and configured to start on boot:

   ```bash
   sudo systemctl start jenkins
   sudo systemctl enable jenkins
   ```

2. **Access Jenkins UI**:
   Open your browser and navigate to `http://<EC2_PUBLIC_IP>:8080`. You’ll be prompted for the **Unlock Jenkins** password.

3. **Find Jenkins Unlock Key**:
   Retrieve the unlock password from the following file:

   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```

   Copy the password and paste it into the Jenkins UI to unlock Jenkins.

4. **Install Recommended Plugins**:
   After logging in, Jenkins will prompt you to install **recommended plugins**. Select **Install suggested plugins** to get started.

5. **Create an Admin User**:
   Once the plugins are installed, Jenkins will prompt you to create an **admin user**. Fill in the required fields (username, password, etc.).

---

### 🛠️ **2. Configuring Jenkins to Integrate with GitHub**

Jenkins needs to connect with your GitHub repository to automate the build process when code changes are pushed.

#### Steps:

1. **Install GitHub Plugin**:

   * Go to the **Manage Jenkins** page.
   * Select **Manage Plugins** > **Available** tab.
   * Search for **GitHub Integration Plugin** and install it.

2. **Configure GitHub Credentials**:

   * Navigate to **Manage Jenkins** > **Manage Credentials** > **(Global)** > **Add Credentials**.
   * Select **Kind**: `Username with password`.
   * **Username**: Your GitHub username.
   * **Password**: Your GitHub **Personal Access Token** (PAT). Generate it under **GitHub Settings** > **Developer settings** > **Personal access tokens**.
   * Name the credential as `GitHub-Personal-Access-Token`.

3. **Set GitHub Repository URL**:

   * Create a new Jenkins job: **New Item** > **Freestyle project**.
   * In the **Source Code Management** section, select **Git**.
   * Enter your GitHub repository URL (e.g., `https://github.com/yourusername/your-repo.git`).
   * Under **Credentials**, select the `GitHub-Personal-Access-Token` credential.

4. **Set Up Webhooks in GitHub**:

   * Navigate to your GitHub repository.
   * Go to **Settings** > **Webhooks** > **Add webhook**.
   * Set the **Payload URL** to `http://<EC2_PUBLIC_IP>:8080/github-webhook/`.
   * Select **Content type**: `application/json`.
   * Choose the event: **Just the push event**.

---

### 🛠️ **3. Setting Up Jenkins Jobs for CI/CD**

With Jenkins now integrated with GitHub, let’s create jobs to automate the build, test, and deployment processes.

#### A. **Build Job (Maven Build)**

1. **Create a New Jenkins Job**:

   * Go to the Jenkins dashboard and create a new **Freestyle project** called `Maven-Build`.

2. **Configure Build Steps**:

   * Under **Build** > **Add build step** > **Invoke top-level Maven targets**.
   * Set **Goals** to: `clean install`.
   * Set **POM** to: `pom.xml` (the location of your Maven POM file).

   This step will clean the project and build the application (e.g., generating a `.jar` or `.war` file).

3. **Set Build Triggers**:

   * Under **Build Triggers**, check **GitHub hook trigger for GITScm polling**.
   * This will ensure Jenkins triggers the build whenever there’s a new commit in the GitHub repository.

#### B. **Test Job (SonarQube Static Analysis)**

To integrate **SonarQube** for static code analysis, we can create a job that runs SonarQube after the build job.

1. **Create a New Jenkins Job**:

   * Name it `SonarQube-Test`.

2. **Configure Build Steps**:

   * Under **Build** > **Add build step** > **Invoke SonarQube scanner**.
   * Select the **SonarQube server** (make sure SonarQube is set up in Jenkins under **Manage Jenkins** > **Configure System** > **SonarQube servers**).
   * Set the **SonarQube properties** (project key, name, etc.).

3. **Set Job Triggers**:

   * In **Post-build Actions**, select **Build other projects** and specify the `Maven-Build` job.
   * This will trigger the SonarQube analysis after the build completes successfully.

#### C. **Deploy Job (Docker + Kubernetes)**

Finally, we’ll set up a deploy job to build Docker images and deploy them to **K3s**.

1. **Create a New Jenkins Job**:

   * Name it `Deploy-to-K3s`.

2. **Configure Build Steps**:

   * **Docker Build**: Under **Build** > **Add build step** > **Execute shell**, add the following commands to build and push the Docker image:

     ```bash
     docker build -t your_image_name .
     docker tag your_image_name:latest your_dockerhub_username/your_image_name:latest
     docker push your_dockerhub_username/your_image_name:latest
     ```

   * **K3s Deployment**: After the Docker image is pushed, use `kubectl` to deploy it to K3s:

     ```bash
     kubectl set image deployment/your-deployment your-container=your_dockerhub_username/your_image_name:latest
     ```

3. **Set Job Triggers**:

   * Configure this job to run after the SonarQube analysis job completes.

---

### 📡 **4. Monitoring Jenkins Builds and Deployments**

Once the jobs are set up, Jenkins will automatically build, test, and deploy your application whenever code is pushed to GitHub.

You can monitor Jenkins through its web UI to track build statuses, view logs, and check for any failures. For advanced monitoring, you can integrate **Prometheus** with Jenkins to collect metrics on builds and deployments.

---

### 🔜 **Next Steps**

Now that Jenkins is set up to automate the CI/CD pipeline, we can move on to integrating **Prometheus** and **Grafana** for advanced monitoring and visualization.
