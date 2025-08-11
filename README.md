# Full Devops Project with Zomato Clone

![All-Ports](https://github.com/herrry107/zomato-devops-project/blob/main/images/pipeline-all-stages.png)

![Grafana-Node-Exporter-Dashboard](https://github.com/herrry107/zomato-devops-project/blob/main/images/grafana-node-exporter-dashboard.png)

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
<pre><code>apt update -y</code></pre>

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

# Configure  Tools

[Go to Configuration tool Page to setup](https://github.com/herrry107/zomato-devops-project/blob/main/Configure-tools/README.md)

# Monitoring of Application

[Go to Monitoring Page to setup Grafana, Prometheus](https://github.com/herrry107/zomato-devops-project/blob/main/Monitoring/README.md)

# Creation EKS Cluster 

We need to run same application on K8S cluster. In order to do that we need to create K8S cluster. I will create the cluster using EKS service in AWS using VS code editor.

**Prerequirements: AWSCLI, KUBECTL, EKSCTL**

[Install awscli,kubectl,eksctl](https://github.com/herrry107/Kubernetes/blob/main/eks-project1/prerequisites.md)

<pre><code>aws configure</code></pre>

**create eks cluster without node-group**

<pre><code>eksctl create cluster --name=pratikcluster --region=ap-south-1 --zones=ap-south-1a,ap-south-1b --without-nodegroup</code></pre>

![EKS-Cluster](https://github.com/herrry107/zomato-devops-project/blob/main/images/eksctl-create-cluster.png)

it will take atleast 20 minutes

get list of cluster
<pre><code>eksctl get cluster</code></pre>

**Create & Associate IAM OIDC Provider for our EKS Cluster**

To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider. To do so using eksctl we can use the below commands.

<pre><code>eksctl utils associate-iam-oidc-provider --region=ap-south-1 --cluster pratikcluster --approve</code></pre>

![EKS-OIDC](https://github.com/herrry107/zomato-devops-project/blob/main/images/eksctl-iam-oidc-attach.png)

create Node Group with additional Add-Ons in Public Subnets, These add-ons will create the respective IAM policies for us automatically within our Node Group role.
<pre><code>eksctl create nodegroup --cluster pratikcluster --name pratik-ng-public --region ap-south-1 --node-type t3.medium  --nodes 2 --nodes-min 1  --nodes-max 2 --node-volume-size=20 --ssh-access --ssh-public-key ubuntu --managed --asg-access --external-dns-access --full-ecr-access --appmesh-access --alb-ingress-access</code></pre>

if need to delete  nodegroup
<pre><code>eksctl delete nodegroup --cluster pratikcluster --name pratik-ng-public --region ap-south-1</code></pre>

if need to delete cluster
<pre><code>eksctl delete cluster pratikcluster</code></pre>

Lets us deploy the same application in the EKS cluster also

To check the nodes creation, go to EKS Service in aws -> Click on the Cluster created -> 'Compute' tab -> Refresh the page if you don't see the node group (1 node group should be seen). Before doing this, make sure in vs code editor the nodes and node groups are created or not.

You can also see the node creation in vs editor by executing -> kubectl get nodes -> You can see 2 nodes.

# Argo CD installation

Inorder to monitor k8s with Prometheus, we need to install ArgoCD. Lets do that Execute the below commands

<pre><code>kubectl create namespace argocd</code></pre>
<pre><code>kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml</code></pre>

wait for sometime till the namespace gets created.
The above command will create a namespace with "argocd" name

By default the argo CD server is not publicly exposed, so we need to expose it publicly. To do that, execute the below command
<pre><code>kubectl patch svc argocd-server -n argocd -p '{"spec":{"type":"LoadBalancer"}}'</code></pre>

After successful execution you should see "patched"

<pre><code>kubectl get pod -n argocd</code></pre>

![EKS-pods](https://github.com/herrry107/zomato-devops-project/blob/main/images/kubectl-argocd-get-pods.png)

wait for 5 minute for the load balancer creation. Once the loadbalancer is created, we will get the load balancer url.

# Monitor Kubernetes with Prometheus

Used to monitor Kubernetes cluster
Additionally, you'll install the node exporter using Helm to collect metrics from your cluster nodes.

Install Node Exporter using Helm

To begin monitoring your Kubernetes cluster, you'll need install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:

Add the Prometheus Community Helm repository:
<pre><code>helm repo add prometheus-community https://prometheus-community.github.io/helm-charts</code></pre>

Create a Kubernetes namespace for the Node Exporter
<pre><code>kubectl create namespace prometheus-node-exporter</code></pre>

Install the Node Exporter using Helm
<pre><code>helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter</code></pre>

Lets continue with load balancer thing of previous step;
<pre><code>export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname'`</code></pre>

To get the loadbalancer url
<pre><code>echo $ARGOCD_SERVER</code></pre>

same command in windows: **echo $env:ARGOCD_SERVER**

copy that url and hit into browser click on advance -> safe and continue -> You will see argoCD

username: admin

for password we need to execute command
<pre><code>export ARGO_PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`</code></pre>
<pre><code>echo $ARGO_PWD</code></pre>

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd1.png)

In the ArgoCD homepage, in the left pane, click on "Manage your repos, projects, settings" icon -> Click on "Repositories" -> Click on "Connect repo using HTTPS" -> Type: git, Project: default, Repo URL: Github-Url-Of-ZomatoProject -> Click on "Connect" (Top bar, left side) -> You can see "connection status " as "successful"

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd2.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd3.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd4.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd5.png)

In the ArgoCD homepage, in the left pane, click on "Manage your apps, and diagnose health problems" -> Click on "New App" -> App Name: zomato, Project Name: default, Sync Policy: Automatic, Repo URL:  Github-Url-Of-ZomatoProject , Revision: HEAD, Path: Give-the-name-of-kubernetes-folder-in-zomato-repo, Cluster URL: select-the-one-from-drop-down-menu, Namespace: default -> Create -> wait till you see the status to completed (it will take 5-10 minutes)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd1.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd6.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd7.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd8.png)

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd9.png)

NOTE: In the repo, in Kubernetes folder, in the deployment.yml file, in the container section  cange the dockerhub username

Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml

Update your Prometheus configuration (prometheus.yml) to add a new job for scrapping metrics from nodeip:9001/metrics. You can do this by adding following configuration to your prometheus.yml file.

Go to the monitoring server

Goto EkS in AWS -> Click on EKS cluster -> Compute tab -> Nodes -> Click on any node -> instance id -> copy the public ip

Go to file sudo vi /etc/prometheus/prometheus.yml

<pre><code>
  - job_name: "k8s"
    metrics_path: "/metrics"
    static_configs:
      - targets: ["nodeIP:9100"]
</code></pre>

node ip is we copied from EKS node

check syntax is  correct or not
<pre><code>promtool check config /etc/prometheus/prometheus.yml</code></pre>
<pre><code>curl -X POST http://localhost:9090/-/reload</code></pre>

allow 9100 and 30001 in security group of custer nodegroup

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/k8s-prometheus.png)

Goto Prometheus and reload. Goto ArgoCD and reload to see whether the pipeline is done or not

![Grafana-argocd](https://github.com/herrry107/zomato-devops-project/blob/main/images/argocd10.png)

copy the public ip of "nodeip" and paste in browser with :30001 port no our application is running.

after all thing please delete nodegroup and cluster also.

if need to delete  nodegroup
<pre><code>eksctl delete nodegroup --cluster pratikcluster --name pratik-ng-public1 --region ap-south-1</code></pre>

if need to delete cluster
<pre><code>eksctl delete cluster pratikcluster</code></pre>

