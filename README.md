# Linux System Health Dashboard
This project involves setting up a comprehensive system health monitoring dashboard for Linux servers using Grafana and Prometheus. The goal is to monitor key metrics (CPU), memory, disk, and network usage), configure alerts for high resource utilization, and provide a user-friendly real-time dashboard.

### Steps to Complete the Project:
1. **Install and Configure Prometheus and Node Exporter**.
2. **Install and Configure Grafana**.
3. **Integrate CPU, Memory, Disk, and Network Usage Metrics**.
4. **Set Up Alerting for High Utilization**.
5. **Create a Real-Time Monitoring Dashboard**.


### Step 1: Install and Configure Prometheus and Node Exporter

Prometheus is a time-series database used to scrape metrics from endpoints, and Node Exporter is a tool that provides Linux system metrics for Prometheus.


#### Step 1.1: Install Prometheus

1. Download and extract Prometheus:

   ```bash
   wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
   tar -xvf prometheus-2.45.0.linux-amd64.tar.gz
   cd prometheus-2.45.0.linux-amd64
   ```

2. Move binaries and set permissions:

   ```bash
   sudo mv prometheus /usr/local/bin/
   sudo mv promtool /usr/local/bin/
   sudo mkdir -p /etc/prometheus /var/lib/prometheus
   sudo mv prometheus.yml /etc/prometheus/
   ```

3. Create a systemd service for Prometheus:

   ```bash
   sudo nano /etc/systemd/system/prometheus.service
   ```

   Add the following content:

   ```ini
   [Unit]
   Description=Prometheus
   After=network.target

   [Service]
   User=prometheus
   ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/var/lib/prometheus/

   [Install]
   WantedBy=multi-user.target
   ```

4. Start and enable Prometheus:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable prometheus
   sudo systemctl start prometheus
   ```

5. Verify Prometheus is running:  
   Open a browser and go to `http://<server-ip>:9090`.

#### Step 1.2: Install Node Exporter

1. Download and extract Node Exporter:

   ```bash
   wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
   tar -xvf node_exporter-1.6.0.linux-amd64.tar.gz
   cd node_exporter-1.6.0.linux-amd64
   ```

2. Move the binary and set permissions:

   ```bash
   sudo mv node_exporter /usr/local/bin/
   ```

3. Create a systemd service for Node Exporter:

   ```bash
   sudo nano /etc/systemd/system/node_exporter.service
   ```

   Add the following content:

   ```ini
   [Unit]
   Description=Node Exporter
   After=network.target

   [Service]
   User=node_exporter
   ExecStart=/usr/local/bin/node_exporter

   [Install]
   WantedBy=multi-user.target
   ```

4. Start and enable Node Exporter:

   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable node_exporter
   sudo systemctl start node_exporter
   ```

5. Verify Node Exporter:  
   Open a browser and go to `http://<server-ip>:9100/metrics`.


#### Step 1.3: Configure Prometheus to Scrape Node Exporter Metrics

1. Edit the Prometheus configuration file:

   ```bash
   sudo nano /etc/prometheus/prometheus.yml
   ```

2. Add Node Exporter under the `scrape_configs` section:

   ```yaml
   scrape_configs:
     - job_name: "node_exporter"
       static_configs:
         - targets: ["localhost:9100"]
   ```

3. Restart Prometheus:

   ```bash
   sudo systemctl restart prometheus
   ```

4. Verify the metrics:  
   Open `http://<server-ip>:9090` and query `node_cpu_seconds_total`.


### Step 2: Install and Configure Grafana

Grafana visualizes metrics from Prometheus in an interactive dashboard.


#### Step 2.1: Install Grafana

1. Add the Grafana repository:

   ```bash
   sudo apt-get install -y software-properties-common
   sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
   wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
   sudo apt update
   ```

2. Install Grafana:

   ```bash
   sudo apt install grafana -y
   ```

3. Start and enable Grafana:

   ```bash
   sudo systemctl enable grafana-server
   sudo systemctl start grafana-server
   ```

4. Access Grafana:  
   Open a browser and navigate to `http://<server-ip>:3000`. Default credentials are **admin/admin**.


#### Step 2.2: Connect Grafana to Prometheus

1. Log in to Grafana and navigate to **Configuration > Data Sources**.
2. Add a new data source and select **Prometheus**.
3. Enter the Prometheus URL: `http://<server-ip>:9090` and click **Save & Test**.


### Step 3: Integrate CPU, Memory, Disk, and Network Usage Metrics



#### Step 3.1: Add Predefined Dashboards

1. Go to **Create > Import** in Grafana.
2. Use an existing dashboard ID from [Grafana’s Dashboard Repository](https://grafana.com/grafana/dashboards/), such as:
   - **1860** for Node Exporter Full Dashboard.
   - **8919** for System Monitoring.

3. Paste the dashboard ID and click **Load**.
4. Select Prometheus as the data source and click **Import**.



#### Step 3.2: Create a Custom Dashboard (Optional)

1. Go to **Create > Dashboard**.
2. Add a new panel for metrics like:
   - CPU Usage: `rate(node_cpu_seconds_total{mode!="idle"}[1m]) * 100`
   - Memory Usage: `node_memory_Active_bytes / node_memory_MemTotal_bytes * 100`
   - Disk Usage: `node_filesystem_avail_bytes / node_filesystem_size_bytes * 100`
   - Network Traffic: `rate(node_network_receive_bytes_total[1m])`.

3. Save the dashboard.

### Step 4: Set Up Alerting for High Utilization

Alerts notify the team when a threshold is exceeded.


#### Step 4.1: Configure Alerts in Prometheus

1. Edit the `prometheus.yml` file to define alert rules:

   ```yaml
   rule_files:
     - "alert.rules"
   ```

2. Create an `alert.rules` file:

   ```bash
   sudo nano /etc/prometheus/alert.rules
   ```

   Add the following example alert:

   ```yaml
   groups:
   - name: Node Alerts
     rules:
     - alert: HighCPUUsage
       expr: rate(node_cpu_seconds_total{mode!="idle"}[1m]) > 0.8
       for: 2m
       labels:
         severity: warning
       annotations:
         summary: "High CPU usage detected"
         description: "CPU usage is above 80% for more than 2 minutes."
   ```

3. Restart Prometheus:

   ```bash
   sudo systemctl restart prometheus
   ```


#### Step 4.2: Configure Alerts in Grafana

1. Open the dashboard and click on a panel.
2. Click **Edit > Alert > Create Alert**.
3. Set the query, condition, and threshold.
4. Configure a notification channel (e.g., email, Slack).


### Step 5: Create a Real-Time Monitoring Dashboard

1. Add relevant metrics panels to a new Grafana dashboard.
2. Organize panels (e.g., CPU on top, memory and disk in the middle, network at the bottom).
3. Save the dashboard and share it with the team.


### Final Step: Document the Setup

Provide detailed documentation for the team, including:

1. **Overview**:
   - Tools used (Prometheus, Grafana, Node Exporter).
   - Purpose of the dashboard.

2. **Installation**:
   - Installation commands for Prometheus, Node Exporter, and Grafana.

3. **Configuration**:
   - Prometheus and Node Exporter configuration.
   - Grafana data source setup.

4. **Monitoring**:
   - Key metrics to monitor (CPU, memory, disk, network).

5. **Alerting**:
   - Alert rules and notification channels.

6. **Maintenance**:
   - How to add new servers or metrics.

By following this step-by-step guide, you’ll successfully implement a robust Linux System Health Dashboard and be equipped to train your team effectively.
