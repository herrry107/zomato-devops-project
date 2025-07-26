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

**4) Install Jenkins**

create jenkins-install.sh
<pre><code>
#install java jdk-17
sudo apt update -y
sudo apt install fontconfig openjdk-17-jre -y
java -version

#install jenkins by official site
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
sudo apt-get update
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
</code></pre>

copy password for first time jenkins-login
<pre><code>cat /var/lib/jenkins/secrets/initialAdminPassword</code></pre>

**5) Install Docker on Ubuntu**

<pre><code>
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
</code></pre>

<pre><code>
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg
</code></pre>

<pre><code>
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
</code></pre>

<pre><code>
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
</code></pre>

<pre><code>
sudo usermod -aG docker ubuntu
sudo chmod 777 /var/run/docker.sock
newgrp docker
sudo systemctl start docker
sudo systemctl status docker
</code></pre>



  


