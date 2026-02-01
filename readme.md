
# Trend-Store Application

## Prerequisites
- AWS Account  
- GitHub Repository: `https://github.com/Vennilavan12/Trend.git` 
---

## Stage 1: Launch EC2 Instance and Run App Locally

1. **Launch EC2 Instance**  
   - Go to AWS Console → EC2 → Launch Instance  
   - Choose Ubuntu 20.04 LTS, t3.micro (Free Tier)  
   - Configure security group to allow SSH (22) and HTTP (80, 8080)  
   - Launch and connect via SSH  

2. **Install Docker on EC2**
   ```
   sudo apt update
   sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   sudo apt update
   sudo apt install docker-ce docker-ce-cli containerd.io
   sudo systemctl start docker
   sudo systemctl enable docker
   sudo usermod -aG docker ubuntu
   ```

3. **Check Docker Version**
   ```
   docker --version
   ```

4. **Clone the Project Repository**
   ```
   git clone https://github.com/Vennilavan12/Trend.git
   cd Trend
   ```

5. **Build and Run Docker Image**
   ```
   docker build -t trendstore .
   docker run -d -p 8080:80 trendstore
   ```

6. **Check Running Container**
   ```
   docker ps
   ```

7. **Expose Port in Security Group**  
   - Add inbound rule: Custom TCP, Port 8080, Source 0.0.0.0/0  

8. **Access the Application**  
   ```
   http://<public-ip>:8080
   ```

---

## Stage 2: Terraform Setup

Install Terraform:
```
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install terraform
terraform -v
```

Install AWS CLI v2:
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws --version
aws configure
```

Run Terraform commands to create EC2 with Jenkins, VPC, IAM role:
```
terraform init
terraform validate
terraform fmt
terraform plan
terraform apply
```

---

## Stage 3: DockerHub

- Create a repository in DockerHub for storing your Docker images.

---

## Stage 4: Kubernetes (EKS Setup)

### IAM
- Create a user `eks-admin` with AdministratorAccess  
- Generate Access Key and Secret Access Key  

### Install AWS CLI v2
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```

### Install kubectl
```
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client
```

### Install eksctl
```
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Setup EKS Cluster
```
eksctl create cluster --name yash-cluster --region us-east-1 --node-type t3.small --nodes-min 2 --nodes-max 2
```

### Check Cluster Status
```
aws eks describe-cluster --name yash-cluster --region us-east-1 --query "cluster.status" --output text
```

### Deploy to Kubernetes
```
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
kubectl get deployments
kubectl get pods
kubectl get svc
kubectl get nodes
```
- This are the commnds to execute the kubernetes pods but here we are executing through jenkins pipeline.

---

## Stage 5: Version Control
- Push code github repo
```
git init
git add .
git commit -m "add"
git remote -v
git remote remove https://github.com/Vennilavan12/Trend.git
git remote remove origin
git remote add origin https://github.com/shreya-yeole/Trend.git
git push origin main
```

---

## Stage 6: Jenkins Setup

Install Jenkins:
```
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Access Jenkins:
```
http://18.215.143.168:8080
```

Retrieve Jenkins initial password:
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Configure plugins:
- AWS Integration  
- GitHub Integration  
- Stage View Plugin  

Setup pipeline with SCM → GitHub repo → credentials → webhook integration.
- after webhok integation you will commit any changes to main repo the builld in jenkins will be automatically trigerred.
---

## Stage 7: Monitoring (Prometheus & Grafana)

Install Helm:
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

Add Helm repositories:
```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

Create monitoring namespace:
```
kubectl create namespace monitoring
```

Install Prometheus:
```
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --set server.service.type=LoadBalancer \
  --set server.service.port=9091
```

Install Grafana:
```
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set service.type=LoadBalancer \
  --set service.port=9092
```

Access Grafana:
```
http://af68b3fec80cf495383fa8a6099d7bca-662286062.us-east-1.elb.amazonaws.com:9092
```

Access Prometheus:
```
http://af68b3fec80cf495383fa8a6099d7bca-662286062.us-east-1.elb.amazonaws.com:9091
```

---

## Application Deployed

Kubernetes LoadBalancer ARN:
```
arn:aws:elasticloadbalancing:us-east-1:031332257575:loadbalancer/a2c20b31bd02c4c0a8b88da47cfb93cc
```

---

