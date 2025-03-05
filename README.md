Production-Level Blue-Green Deployment CI/CD Pipeline

Scenario

Upgrade an application with new features while ensuring zero downtime by redirecting traffic from the blue to the green deployment. In case of issues, revert changes from Green back to Blue.

Tools Used

EKS Cluster Deployment using Terraform

SonarQube (Code Quality and Security Analysis)

Nexus (Artifact Repository)

Jenkins (CI/CD Pipeline)

Implementation Walkthrough

Select Deployment Environment

Ensure traffic switch capabilities are configured.

Define security group port settings.

Select Environment in which we want to deploy and make sure to check switch traffic.

Security Group Port:

Step 1: Create Master Server (Ubuntu) for EKS Cluster

Update System

sudo apt update

Establish AWS CLI Connection

Install Terraform

sudo snap install terraform --classic

Clone Repository

git clone https://github.com/jaiswaladi246/Blue-Green-Deployment.git

Initialize and Apply Terraform

cd Blue-Green-Deployment/cluster
terraform init
terraform apply -auto-approve

Install kubectl

Configure EKS Cluster Connection

aws eks --region ap-south-1 update-kubeconfig --name <cluster-name>

Set Up RBAC for Cluster Access

Add Jenkins Secret Token for Authentication

Go to Manage Jenkins → Credentials → Secret Text

Step 2: Create Jenkins Server

Update System

sudo apt update

Install Java

sudo apt install openjdk-17-jre-headless -y

Install Jenkins from the official site

Enable Jenkins Service

Install Required Tools on Jenkins

Docker

sudo apt install docker.io -y
sudo usermod -aG docker jenkins

Trivy (Security Scanning)

kubectl

Install Jenkins Plugins

SonarQube Scanner, Maven, Docker Pipeline, Kubernetes Plugins

Configure Jenkins Pipeline – Blue-Green

Pipeline Setup:

Discard old builds (max 2)

Configure Maven & SonarQube

Credentials setup:

Token for SonarQube

Need to add SonarQube server inside Jenkins

Manage Jenkins → System → SonarQube Installation

Manage Jenkins → Managed Files → Global Maven settings → Add credentials for Nexus server (maven release and maven snapshot)

Update Maven release and Maven snapshot URL in pom.xml

Stages:

Add tools Maven inside tools section

Stage 1 - Git checkout source code

Stage 2 - Compile

mvn compile

Stage 3 - Test

mvn test -DskipTests=true

Stage 4 - Trivy file system scan

Stage 5 - SonarQube scan

SonarQube server to publish report

Scanner to perform analysis and generate a report on the server

Define SonarQube server under the environment section

Stage 6 - Perform Code Quality Check

Name: Quality Gate Check

Copy Jenkins URL → Go to SonarQube → Administration → Configuration → Webhooks

Pipeline syntax for timeout: 1 hour

Build Stage

mvn package -DskipTests=true

Publish artifact to Nexus

Create Docker images for Blue and Green environments

Stage 7 - Docker Build & Tag Image

Stage 8 - Docker Image Scan Stage

Stage 9 - Push Image to Docker Hub Repo

Important Change: SQL server and LB are not going to change

Deploy MySQL Deployment and Service

Deploy MySQL as a backend database in Kubernetes.

Deploy Application Load Balancer Service

Setup Load Balancer to manage traffic between Blue and Green environments.

Blue-Green Traffic Switching

Update the Kubernetes service to switch traffic from Blue to Green.

In case of issues, revert back to the Blue deployment.

Complete Jenkins file in the code for reference

Step 3: Create Nexus Server

Update System

sudo apt update

Setup Nexus using Docker container

sudo apt install docker.io -y
sudo usermod -aG docker ubuntu

Restart machine or run:

newgrp docker

Run Nexus container

docker run -d -p 8081:8081 sonatype/nexus3

Retrieve Admin Credentials

docker exec -it <container-id> /bin/bash
cd sonatype-work/nexus3
cat password

Step 4: Create SonarQube Server

Update System

sudo apt update

Setup SonarQube using Docker container

sudo apt install docker.io -y
sudo usermod -aG docker ubuntu

Restart machine or run:

newgrp docker

Run SonarQube container

docker run -d -p 9000:9000 sonarqube:lts/community

Default username and password:

Username: admin

Password: admin
