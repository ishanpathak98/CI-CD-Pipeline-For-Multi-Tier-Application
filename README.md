# CI/CD Pipeline for Multi-Tier Application

This repository contains the code and configuration files necessary to set up a CI/CD pipeline for a multi-tier application. The application consists of a frontend and backend, both of which are containerized using Docker and deployed using Kubernetes on AWS EKS.

## Repository Link

[GitHub Repository](https://github.com/ishanpathak98/CI-CD-Pipeline-For-Multi-Tier-Application.git)

## Prerequisites

- AWS CLI installed and configured
- Docker installed
- Kubernetes tools (`kubectl` and `eksctl`) installed

## Commands Used
1. ## Update system packages
 ```bash
   sudo apt-get update
  ```

2. ## Clone the GitHub Repository
 ```bash
git clone https://github.com/ishanpathak98/CI-CD-Pipeline-For-Multi-Tier-Application.git
cd CI-CD-Pipeline-For-Multi-Tier-Application/
```

## Navigate to Application Code

```bash
cd Application-Code/
```

## Install Docker

```bash

sudo apt install docker.io
sudo usermod -aG docker $USER
sudo chown $USER /var/run/docker.sock
```
## Build Docker Images

```bash

cd backend/
docker build -t three-tier-backend .
cd ../frontend/
docker build -t three-tier-frontend .
```
## Run Docker Containers

To run the Docker containers for the frontend and backend:

```bash
docker run -d -p 3000:3000 three-tier-frontend:latest
docker run -d -p 8080:8080 three-tier-backend:latest
```

## Install AWS CLI

```bash

curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install -i /usr/local/aws-cli -b /usr/local/bin --update
aws configure
```
## Push Docker Images to AWS ECR

```bash

aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/b7m8i4f8
docker tag three-tier-frontend:latest public.ecr.aws/b7m8i4f8/three-tier-frontend:latest
docker push public.ecr.aws/b7m8i4f8/three-tier-frontend:latest
```
## Install Kubernetes Tools

```bash

curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin
kubectl version --short --client

curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```
## Create EKS Cluster

```bash

eksctl create cluster --name three-tier-cluster --region us-west-2 --node-type t2.medium --nodes-min 2 --nodes-max 2
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
```
## Deploy Kubernetes Resources

```bash

kubectl create namespace workshop
kubectl apply -f deploy.yaml -n workshop
kubectl get pods -n workshop
```
## Integrate MongoDB with the Application

   Create MongoDB Deployment and Service
   Ensure that MongoDB is deployed within your Kubernetes cluster or accessible externally. You can create a Kubernetes Deployment and Service for MongoDB if it’s not already deployed:

    yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
  namespace: workshop
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:latest
        ports:
        - containerPort: 27017
        env:
        - name: MONGO_INITDB_ROOT_USERNAME
          value: root
        - name: MONGO_INITDB_ROOT_PASSWORD
          value: example
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  namespace: workshop
spec:
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017

Update Backend Configuration
Modify the backend configuration to connect to the MongoDB service. Update the MongoDB connection string in your backend application code to point to the MongoDB service:

javascript

const mongoose = require('mongoose');
const dbURI = 'mongodb://root:example@mongodb-service:27017/mydatabase?authSource=admin';

mongoose.connect(dbURI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.log(err));

Deploy the Backend with MongoDB Integration
Redeploy the backend service to ensure it connects to the MongoDB instance:

bash

        kubectl apply -f backend-deployment.yaml -n workshop

Dockerfile for Frontend

Dockerfile
```
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```
Dockerfile for Backend

Dockerfile
```
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

Testing the Application

    Access the Frontend:
        Use the public IP of the EC2 instance and the port 3000 to test the frontend.
        Example: http://<EC2-PUBLIC-IP>:3000

    Access the Backend:
        The backend can be accessed on port 8080.
        Example: http://<EC2-PUBLIC-IP>:8080

    Access MongoDB:
        The backend should be able to interact with MongoDB through the internal service mongodb-service within the Kubernetes cluster.


