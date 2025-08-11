# Argo CD Installation

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
