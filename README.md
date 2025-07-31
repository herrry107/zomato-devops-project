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

**12) Build Pipeline**

New Item -> Pipeline
<pre><code>
  pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node23'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage ("clean workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git 'https://github.com/herrry107/Zomato-Jenkins-Docker.git'
            }
        }
        stage("Sonarqube Analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato \
                    -Dsonar.projectKey=zomato '''
                }
            }
        }
        stage("Code Quality Gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
            } 
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit -n', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
    }
}
        stage ("Trivy File Scan") {
            steps {
                sh "trivy fs . > trivy.txt"
            }
        }
        stage ("Build Docker Image") {
            steps {
                sh "docker build -t zomato ."
            }
        }
        stage ("Tag & Push to DockerHub") {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh "docker tag zomato herrypr107/zomato:latest "
                        sh "docker push herrypr107/zomato:latest "
                    }
                }
            }
        }
        stage('Docker Scout Image') {
            steps {
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                       sh 'docker-scout quickview herrypr107/zomato:latest'
                       sh 'docker-scout cves herrypr107/zomato:latest'
                       sh 'docker-scout recommendations herrypr107/zomato:latest'
                   }
                }
            }
        }
        stage ("Deploy to Container") {
            steps {
                sh 'docker run -d --name zomato -p 3000:3000 herrypr107/zomato:latest'
            }
        }
    }
    post {
    always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: """
                <html>
                <body>
                    <div style="background-color: #FFA07A; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: white; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                </body>
                </html>
            """,
            to: 'pratikumar13@gmail.com',
            mimeType: 'text/html',
            attachmentsPattern: 'trivy.txt'
        }
    }
}
</code></pre>

save and apply 

now run command in server
<pre><code>
#add jenkins user into docker  group  
sudo usermod -aG docker jenkins
</code></pre>

<pre><code>
#restart jenkins service
sudo systemctl restart jenkins
</code></pre>

build now

# Monitoring of Application

**1) Launch an Instance with name Monitoring-Server (Ubuntu 24.04, t2.large,30GB)**

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

**2) Configure Prometheus Plugin Integration**

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

<monitoring-server-ip>:3000 // username: admin, password: admin

You  will see **Grafana Dashboard**

The first thing that we have to do in Grafana is to add the data source Lets add the data source;

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana_data_source.png)

![Grafana-Data-Source](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana_data_source1.png)
