# Full Devops Project with Zomato Clone

**1) Launch an Instance (Ubuntu 24.04, t2.large,30GB)**

1.1) Open the below Port No
- HTTPS 443 AnyWhereIPv4
- HTTP 80 AnyWhereIPv4
- SSH 22 AnyWhereIPv4
- Custom TCP 3000 AnyWhereIPv4 --> **App and Grafana Port**
- Custom TCP 8080 AnyWhereIPv4 --> **Jenkins**
- Custom TCP 9090 AnyWhereIPv4 --> **Prometheus**
- Custom TCP 9000 AnyWhereIPv4 --> **SonarQube**
- Custom TCP 9100 AnyWhereIPv4 --> **Node Export Port (If Grafana and Prometheus need to work, the Node Exporter should push  the CPU, Memory usage and other details, so is why we have opened 9100 Port)**

  


