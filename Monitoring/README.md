# Monitoring with Grafana and Prometheus

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-jenkins-dashboard.png)

# Launch an Instance with name Monitoring-Server (Ubuntu 24.04, t2.large,30GB)

We will install Grafana, Prometheus, Node Exporter in the above instance and then we will monitor

First Create a dedicated Linux user for Prometheus and download Prometheus
<pre><code>sudo useradd --system --no-create-home --shell /bin/false prometheus</code></pre>

**now download and extract prometheus**
<pre><code>wget https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-amd64.tar.gz</code></pre>
<pre><code>tar -xvf prometheus-3.5.0.linux-amd64.tar.gz</code></pre>
<pre><code>
cd prometheus-3.5.0.linux-amd64
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
</code></pre>

set ownership of directory
<pre><code>
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
</code></pre>

Create a systemd unit configuration file for Prometheus:
<pre><code>sudo vi /etc/systemd/system/prometheus.service</code></pre>

Add the following content to the prometheus.service file:
<pre><code>
[Unit]
Description=Prometheus
Wants=network-onlline.target
After=network-online.target

StartLimitIntervalsec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/data \
    --web.listen-address=0.0.0.0:9090 \
    --web.enable-lifecycle

Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
</code></pre>

Here's a brief explanation of the key parts in this prometheus.service file:

User and Group specify the Linux user and group under which Prometheus will run.

ExecStart is where you specify the Prometheus binary path, the location of the configuration file (prometheus.yml), the storage directory, and other settings.

web.listen-address configures Prometheus to listen on all network interfaces on port 9090.

web.enable-lifecycle allows for management of Prometheus through API calls.

enable and start prometheus
<pre><code>sudo systemctl enable prometheus</code></pre>
<pre><code>sudo systemctl start prometheus</code></pre>

Now Promwtheus in browser using your server's IP and port 9090: http://<your-server-ip>:9090 (if doesn't work remove http)

Click on 'Status' dropdown -> Click on 'Targets' -> You can see 'Prometheus (1/1) up'

![Prometheus](https://github.com/herrry107/zomato-devops-project/blob/main/images/prometheus1.png)

add node_exporter user
<pre><code>
sudo useradd --system --no-create-home --shell /bin/false node_exporter
</code></pre>

**now download and extract prometheus node exporter**
<pre><code>
wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
</code></pre>

**Extract Node Exporter files, move the binary, and clean up
<pre><code>tar -xvf node_exporter-1.9.1.linux-amd64.tar.gz</code></pre>
<pre><code>sudo mv node_exporter-1.9.1.linux-amd64/node_exporter /usr/local/bin/</code></pre>
<pre><code>rm -rf node*</code></pre>

create a systemd unit configuration file for Node Exporter
<pre><code>sudo vi /etc/systemd/system/node_exporter.service</code></pre>
<pre><code>
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalsec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter --collector.logind
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
</code></pre>

Note: Replace --collector.logind with any additional flags as needed.

Enable and Start node exporter
<pre><code>sudo systemctl enable node_exporter</code></pre>
<pre><code>sudo systemctl start node_exporter</code></pre>

# Configure Prometheus Plugin Integration

As of now we created Prometheus service, but we need to add a job in order to fetch the details by node exporter. So for that we need to create 2 jobs, one with 'node exporter' and the  other with 'jenkins' as shown below;

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline

Prometheus Configuration:

To  configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file

The path of prometheus.yml
<pre><code>vi /etc/prometheus/prometheus.yml</code></pre>

add these lines in the end of prometheus.yml
<pre><code>
  - job_name: "node_exporter"
    static_configs:
      - targets: ["monitoring-ip:9090"]
  - job_name: "jenkins"
    metrics_path: "/prometheus"
    static_configs:
      - targets: ["jenkins-ip:8080"]
</code></pre>

Check the validity of the configuration file
<pre><code>promtool check config /etc/prometheus/prometheus.yml</code></pre>

You should see "SUCCESS" when you run the above command

Reload the Prometheus configuration without restarting:
<pre><code>curl -X POST http://localhost:9090/-/reload</code></pre>

Access Prometheus in browser or refresh page
http://<prometheus-ip>:9090/targets

![Prometheus](https://github.com/herrry107/zomato-devops-project/blob/main/images/prometheus_dashboard.png)

You can see 'Prometheus (1/1) up',  'node exporter(1/1) up','Jenkins (1/1 up)'

# Install Grafana

Install Grafana in monitoring server

**1) Install  Dependencies**

<pre><code>sudo apt-get update</code></pre>
<pre><code>sudo apt-get install -y apt-transport-https software-properties-common</code></pre>

Add the GPG Key:
<pre><code>wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -</code></pre>

You should see OK when executed the above command

Add Grafana Repository
<pre><code> echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list</code></pre>

**Update and Install Grafana**
<pre><code>sudo apt-get update</code></pre>
<pre><code>apt-get -y install grafana</code></pre>

**Enable and Start Grafana**
<pre><code>sudo systemctl enable grafana-server</code></pre>
<pre><code>sudo systemctl start grafana-server</code></pre>

now go to browser and type 

monitoring-server-ip:3000 // username: admin, password: admin

You  will see **Grafana Dashboard**

The first thing that we have to do in Grafana is to add the data source Lets add the data source;

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana_data_source.png)

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana_data_source1.png)

# Grafana Dashboard

**Node Exporter Dashboard for Prometheus**

click new dashboard and import 

![Grafana-Node-Exporter-Dashboard](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-import-dashboard.png)

type id for node-exporter which is 1860 and click on Load

![Grafana-Node-Exporter-Dashboard](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-import-dashboard1.png)

click on select a prometheus data source -> Prometheus -> import

![Grafana-Node-Exporter-Dashboard](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-import-dashboard2.png)

![Grafana-Node-Exporter-Dashboard](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-node-exporter-dashboard.png)

now same import for jenkins

**Jenkins Dashboard for Prometheus**

click new dashboard and import 

type id for jenkins which is 9964 and click on Load

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-import-jenkins-dashboard.png)

click on select a prometheus data source -> Prometheus -> import

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-import-jenkins-dashboard1.png)

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-jenkins-dashboard.png)
