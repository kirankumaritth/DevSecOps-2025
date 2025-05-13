# 🚀 **Integrating Prometheus and Grafana for Monitoring**

We'll cover the following steps:

1. **Install Prometheus on your EC2 instance** to collect metrics from Jenkins, K3s, and other services.
2. **Install Grafana** to visualize the data collected by Prometheus.
3. **Configure Prometheus to Scrape Metrics from Jenkins and K3s**.
4. **Set up Grafana Dashboards** to visualize Jenkins build data and K3s cluster performance.

---

### 🛠️ **1. Installing Prometheus on EC2**

Prometheus will collect metrics from Jenkins and K3s, so let’s start by installing Prometheus on your EC2 instance.

#### Steps:

1. **Install Prometheus**:

   Run the following commands to install Prometheus on your EC2 instance:

   ```bash
   sudo apt update
   sudo apt install -y prometheus
   ```

2. **Start Prometheus Service**:
   Start the Prometheus service and ensure it starts on boot:

   ```bash
   sudo systemctl start prometheus
   sudo systemctl enable prometheus
   ```

3. **Verify Prometheus is Running**:
   Open your browser and navigate to `http://<EC2_PUBLIC_IP>:9090`. You should see the Prometheus web UI.

---

### 🛠️ **2. Installing Grafana on EC2**

Grafana will allow us to visualize the data Prometheus collects.

#### Steps:

1. **Install Grafana**:

   Run the following commands to install Grafana:

   ```bash
   sudo apt install -y software-properties-common
   sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
   sudo apt update
   sudo apt install -y grafana
   ```

2. **Start Grafana Service**:
   Start the Grafana service and enable it to start on boot:

   ```bash
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```

3. **Verify Grafana is Running**:
   Open your browser and navigate to `http://<EC2_PUBLIC_IP>:3000`. You should see the Grafana login screen.

   The default credentials are:

   * **Username**: `admin`
   * **Password**: `admin` (You’ll be prompted to change it on first login)

---

### 🛠️ **3. Configuring Prometheus to Scrape Metrics from Jenkins and K3s**

Now, let’s configure Prometheus to collect metrics from Jenkins and K3s.

#### A. **Configure Prometheus to Scrape Jenkins Metrics**

Jenkins exposes metrics in a format Prometheus can scrape through the **Prometheus Metrics Plugin**.

1. **Install the Prometheus Metrics Plugin in Jenkins**:

   * Go to the Jenkins dashboard.
   * Navigate to **Manage Jenkins** > **Manage Plugins** > **Available** tab.
   * Search for **Prometheus Metrics Plugin** and install it.

2. **Configure the Plugin**:

   * After installing the plugin, go to **Manage Jenkins** > **Configure System**.
   * Scroll down to **Prometheus Metrics** and enable the plugin.
   * You’ll see a default endpoint: `http://<JENKINS_PUBLIC_IP>:8080/metrics`.

3. **Add Jenkins to Prometheus Scrape Config**:
   Open the Prometheus configuration file (`/etc/prometheus/prometheus.yml`) and add Jenkins as a scrape target:

   ```yaml
   scrape_configs:
     - job_name: 'jenkins'
       static_configs:
         - targets: ['<JENKINS_PUBLIC_IP>:8080']
   ```

4. **Restart Prometheus**:
   Restart the Prometheus service to apply the changes:

   ```bash
   sudo systemctl restart prometheus
   ```

---

#### B. **Configure Prometheus to Scrape K3s Metrics**

K3s exposes metrics that Prometheus can scrape by default on port `6443`. To enable this:

1. **Enable Prometheus Metrics in K3s**:
   Prometheus metrics are typically exposed via the **Kubelet**. Ensure the following flags are set when starting K3s:

   ```bash
   --kubelet-https=true --kubelet-insecure-tls
   ```

2. **Add K3s to Prometheus Scrape Config**:
   Edit `/etc/prometheus/prometheus.yml` and add K3s as a scrape target:

   ```yaml
   scrape_configs:
     - job_name: 'k3s'
       static_configs:
         - targets: ['<K3S_API_SERVER_IP>:6443']
   ```

3. **Restart Prometheus**:
   Restart Prometheus again to load the new configuration:

   ```bash
   sudo systemctl restart prometheus
   ```

---

### 🛠️ **4. Setting Up Grafana Dashboards**

With Prometheus collecting metrics from Jenkins and K3s, it’s time to set up Grafana to visualize those metrics.

#### A. **Add Prometheus as a Data Source in Grafana**

1. **Log in to Grafana**:
   Navigate to `http://<EC2_PUBLIC_IP>:3000` and log in using the default credentials (`admin` / `admin`).

2. **Add Prometheus Data Source**:

   * In Grafana, click on the **gear icon** in the left sidebar to go to **Configuration**.
   * Click **Data Sources** > **Add data source**.
   * Select **Prometheus** as the data source.
   * Set the **URL** to `http://<EC2_PUBLIC_IP>:9090`.
   * Click **Save & Test** to ensure Grafana can connect to Prometheus.

---

#### B. **Create Grafana Dashboards**

Now, let’s create Grafana dashboards to visualize Jenkins build data and K3s performance.

1. **Create a New Dashboard**:

   * From the Grafana UI, click the **plus icon** in the left sidebar and select **Dashboard**.
   * Click **Add New Panel** to create a new visualization.

2. **Visualize Jenkins Metrics**:

   * In the **Query** section, select **Prometheus** as the data source.

   * For Jenkins, you can query metrics like build durations, success rates, etc. For example, a simple query to show build success rate:

     ```prometheus
     jenkins_job_builds{result="SUCCESS"}
     ```

   * Choose a visualization type, such as a **Time series** or **SingleStat**.

3. **Visualize K3s Metrics**:

   * You can visualize K3s cluster performance using metrics like CPU, memory, and pod status.

   * A sample query to show CPU usage:

     ```prometheus
     rate(container_cpu_usage_seconds_total[5m])
     ```

   * Choose a **Graph** or **Bar chart** visualization to display this data.

4. **Save the Dashboard**:

   * After adding the necessary panels for Jenkins and K3s, click **Save** and give your dashboard a meaningful name.

---

### 📡 **5. Monitoring Jenkins Builds and K3s Cluster Performance**

Now that Prometheus is scraping Jenkins and K3s metrics and Grafana is visualizing them, you can:

* **Monitor Jenkins**: Track build success rates, build times, and failures in real-time.
* **Monitor K3s**: Track the resource usage, pod status, and overall health of your Kubernetes cluster.

You can now visualize the performance of your DevSecOps pipeline and infrastructure, providing you with valuable insights for optimization.

---
