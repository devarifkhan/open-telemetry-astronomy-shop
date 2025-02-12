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

- Type: All Traffic, Protocol: All, Port Range: All, Source: Anywhere-IpV4here

## Now Access the project
```bash
http://<public-ip>:8080
```

## Containerize the OpenTelemetry Astronomy Shop

```bash
cd opentelemetry-demo/src/product-catalog
export PRODUCT_CATALOG_PORT=8088
go build -o product-catalog .
```