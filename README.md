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

![All-Ports](https://github.com/herrry107/zomato-devops-project/blob/main/images/all_ports.png)

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

**6) Install trivy on Ubuntu**
<pre><code>
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy
</code></pre>

<pre><code>
#check trivy installed or not
trivy --version
</code></pre>

**7) Install Docker Scout**

Make  sure to Login to DockerHub account
<pre><code>docker login -u user-name</code></pre>

![docker-login](https://github.com/herrry107/zomato-devops-project/blob/main/images/docker-login.png)

<pre><code>
curl -fsSL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh -o install-scout.sh
sh install-scout.sh
</code></pre>
<pre><code>sudo chmod 777 /var/run/docker.sock</code></pre>

**8) Install SonaQube using Docker**
<pre><code>docker  run -d --name sonar -p 9000:9000 sonarqube:lts-community</code></pre>

Go to browser and type ip:9000 it will ask for  username and password by default both are 'admin'

**9) Install required Plugin into Jenkins**

- Pipeline Stage View  (show pipeline stages)
- Eclipse Temurin installer (when work with different version of java)
- SonarQube Scanner
- Docker,Docker Commons,Docker Pipeline,Docker API,docker-build-step
- Email Extension Template
- OWASP Dependency-CheckVersion
- NodeJS
- Prometheus metrics

![Jenkins-Plugin](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-plugin.png)

**10) Create SonarQube token**

- SonarQube Console -> Administrator -> Security -> Users -> Click on  3 bars under Token -> Give Name and Generate

![SonarQube-Token](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-toke-generate.png)

![SonarQube-Token](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-toke-generate1.png)

**10.1) Tools Configuration in Jenkins**

- Jenkins -> Manage Jenkins -> Tools -> JDK installations -> Add Jdk -> jdk17 -> Install automatically -> jdk-17.0.8.1+1

![JDK-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-jdk.png)

- Jenkins -> Manage Jenkins -> Tools -> Git installations -> Add Git

![Git-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-git.png)

- Jenkins -> Manage Jenkins -> Tools -> SonarQube Scanner installations -> Add SonarQube Scanner -> sonar-scanner -> Install automatically -> Install from Maven Central

![SonarQube-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-sonarqube.png)

- Jenkins -> Manage Jenkins -> Tools -> NodeJS installations -> Add NodeJS -> node23 -> Install automatically -> Install from nodejs.org

![NodeJS-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-nodejs.png)

- Jenkins -> Manage Jenkins -> Tools -> Dependency-Check installations -> Add Dependency-Check -> DP-Check -> Install automatically

![Dependency-Check-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-tools-DP-Check.png)

- Jenkins -> Manage Jenkins -> Tools -> Docker installations -> Add Docker -> docker -> Install automatically -> Download from docker.com -> latest
- 

**10.2) Add SolarQube Credentials in Jenkins**

Manage Jenkins ->  Credentials ->  global-> Add Credentials -> Kind (Secret text) -> secret (token) -> id (any-name)

![SolarQube-Credential](https://github.com/herrry107/zomato-devops-project/blob/main/images/jenkins-sonar-token-secret-credentials.png)

**10.3) Add Dockerhub Credentials in Jenkins**

Manage Jenkins ->  Credentials ->  global-> Add Credentials -> Kind (Username with Password) -> Username (dockerhub-id) -> Password (dockerhub-password) -> id (any-name)

![Docker-Credentials](https://github.com/herrry107/zomato-devops-project/blob/main/images/docker-login-credentials.png)

**10.4) Add Email to get Notification of Success and Failure**

Go to App Password in Security and generate password and save in jenkins credentials (in jenkins credentials remove app password spaces like if password is "ab cd" than "abcd")

![Gmail-App-Password](https://github.com/herrry107/zomato-devops-project/blob/main/images/gmail-app-password.png)

Manage Jenkins ->  Credentials ->  global-> Add Credentials -> Kind (Username with Password) -> Username (gmail address) -> Password (App-password) -> id (email-creds)

![Gmail-App-Password](https://github.com/herrry107/zomato-devops-project/blob/main/images/docker-sonarqube-both-credentials.png)

**11) SonarQube Configure in Jenkins/Manage/System**

Go to Jenkins -> Manage Jenkins -> System -> SonarQube Servers 

![SonarQube-Server-Configure](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-configure-in-jenkins-system.png)

**11.1) Extended E-mail Notification in Jenkins/Manage/System**

Extended E-mail Notification configure for notification 

![Extended E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/Extended-email-notification.png)

**11.2) E-mail Notification in Jenkins/Manage/System (remove app password spaces)**

![E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/e-mail-notification1.png)

![E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/e-mail-notification2.png)

**11.2) Default triggers for email in Jenkins/Manage/System (remove app password spaces)**

![E-mail Notification](https://github.com/herrry107/zomato-devops-project/blob/main/images/notification-default-triggers.png)

**11.3) Create sonarqube-webhooks**'

Go to sonarqube -> Administrator -> Configurations  -> Webhooks

![sonarqube-webhooks](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-webhook1.png)

![sonarqube-webhooks](https://github.com/herrry107/zomato-devops-project/blob/main/images/sonarqube-webhook2.png)
