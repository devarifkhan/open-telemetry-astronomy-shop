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