# Practicing DevOps with OpenTelemetry Astronomy Shop

## Overview

The OpenTelemetry Astronomy Shop is a microservice-based distributed system designed to illustrate the implementation of
OpenTelemetry in a near real-world environment. This repository can be used to practice various DevOps skills, including
continuous integration, continuous deployment, monitoring, and observability.

## Repository

You can find the repository here: [OpenTelemetry Astronomy Shop](https://github.com/open-telemetry/opentelemetry-demo)

## Goals

The primary goals of this project are:

1. **Provide a realistic example** of a distributed system that can be used to demonstrate OpenTelemetry instrumentation
   and observability.
2. **Build a base** for vendors, tooling authors, and others to extend and demonstrate their OpenTelemetry integrations.
3. **Create a living example** for OpenTelemetry contributors to use for testing new versions of the API, SDK, and other
   components or enhancements.

## Documentation

For detailed documentation on how to install and run the demo, and to explore various scenarios to view OpenTelemetry in
action, visit: [OpenTelemetry Demo Documentation](https://opentelemetry.io/docs/demo/)

## Prepare the Environment

## Set up a AWS EC2 instance

- name: devops
- image: Ubuntu Server 24.04 LTS
- instance type: t2.large (2 vCPUs, 8GB RAM)
- key pair: devops.pem
- Allow all traffic from anywhere
- 30 GB Storage

## Connect to the EC2 instance

```bash
$ chmod 400 devops.pem
$ ssh -i devops.pem ubuntu@<public-ip>
```

## Install Docker to the EC2 instance

setup docker `apt` repository

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

## not to use `sudo` every time.

```bash
sudo usermod -aG docker $USER
```

## Install kubectl to the EC2 instance

```bash
# Download the latest release with the command:
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
```

## Validate the binary (optional)

```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
```

## Install kubectl

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

## If you do not have root access on the target system, you can still install kubectl to the ~/.local/bin directory:

```bash
chmod +x kubectl
mkdir -p ~/.local/bin
mv ./kubectl ~/.local/bin/kubectl
# and then append (or prepend) ~/.local/bin to $PATH
```

## Test to ensure the version you installed is up-to-date:

```bash
kubectl version --client
#  use this for detailed view of version:
kubectl version --client --output=yaml
#
```

## Install Terraform to the EC2 instance

```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common

wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null

gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | \
sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update

sudo apt-get install terraform

terraform --version
```

## Run Locally the OpenTelemetry Astronomy Shop Using Docker Compose

```bash
git clone https://github.com/open-telemetry/opentelemetry-demo.git
cd opentelemetry-demo
docker compose up
```

## Setup Security Group

go to the security group edit inbound rules and add the following rules:

- Type: All Traffic, Protocol: All, Port Range: All, Source: Anywhere-IpV4

## Now Access the project
```bash
http://<public-ip>:8080
```

## After Creating the EKS and S3 bucket setup the EKS
```bash
aws eks --region ap-south-1 update-kubeconfig --name my-eks-cluster
kubectl config view
kubectl get nodes
```
Now run all microservice by kubernetes
```bash
~/opentelemetry-demo/kubernetes$ kubectl apply -f opentelemetry-demo.yaml 
```
To access from outside make this type to LoadBalancer
```bash
~/opentelemetry-demo/kubernetes$ kubectl get svc | grep frontendproxy
opentelemetry-demo-frontendproxy           ClusterIP   172.20.11.57     <none>        8080/TCP                                                   6m19s
~/opentelemetry-demo/kubernetes$ kubectl edit svc opentelemetry-demo-frontendproxy 
type: LoadBalancer
```

now you can access the project by the LoadBalancer IP
```bash
http://<EXTERNAL-IP>:8080/
```

## After setup the Ingress Controller
change the type to NodePort
```bash
~/opentelemetry-demo/kubernetes$ kubectl edit svc opentelemetry-demo-frontendproxy
type: NodePort
```
## Now write the ingress file
```bash
vim ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-proxy
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  ingressClassName: alb
  rules:
    - host: example.com
      http:
        paths:
          - path: "/"
            pathType: Prefix
            backend:
                service:
                    name: opentelemetry-demo-frontendproxy
                    port:
                      number: 8080
```

## Now get the kubectl ingress load balancer
```bash

/opentelemetry-demo/kubernetes$ kubectl get ing
NAME             CLASS   HOSTS         ADDRESS                                                                   PORTS   AGE
frontend-proxy   alb     example.com   k8s-default-frontend-6e54782b3e-1502274640.ap-south-1.elb.amazonaws.com   80      27s
```

## Change the etc hosts to map to example.com [ use firefox chrome will not work]
```bash
sudo vim /etc/hosts

```

## Create a setup the domain in Route53 and assign it to the LoadBalancer By Yourself

## Install AgroCD And make pipeline for the project
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```
## Get the username and password for ArgoCD
```bash
# The username is 'admin'
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```