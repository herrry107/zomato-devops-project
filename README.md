# Full Devops Project with Zomato Clone

**1) Launch an Instance with name Zomato-Server (Ubuntu 24.04, t2.large,30GB)**

1.1) Open the below Port No
- HTTPS 443 AnyWhereIPv4
- HTTP 80 AnyWhereIPv4
- SSH 22 AnyWhereIPv4
- Custom TCP 3000 AnyWhereIPv4 --> **App and Grafana Port**
- Custom TCP 8080 AnyWhereIPv4 --> **Jenkins**
- Custom TCP 9090 AnyWhereIPv4 --> **Prometheus**
- Custom TCP 9000 AnyWhereIPv4 --> **SonarQube**
- Custom TCP 9100 AnyWhereIPv4 --> **Node Export Port (If Grafana and Prometheus need to work, the Node Exporter should push  the CPU, Memory usage and other details, so is why we have opened 9100 Port)**

**2) Connect and Update Command in Zomato-Server Instance**
<pre><code>
#switch user to root
sudo su</code></pre>
<pre><code>apt update -yM</code></pre>

**3) Install AWS CLI**
<pre><code>
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
</code></pre>

<pre><code>
#check awscli install or not
aws version
</code></pre>

<pre><code>
#configure aws cli
aws configure
</code></pre>



  


