# Implementation of an End to End DevOps on a Golang web application. We will implement the following things 

 - Containerization (Multi Stage Docker Build)
 - Creating Kubernetes Manifests
 - Kubernetes Cluster creation and setup
 - Continuous Integration using GitHub Actions
 - Continuous Delivery using Argo CD
 - Helm chart creation and configuration for multiple environments
 - Ingress controller creation, configuration to expose application
 - DNS mapping for our domain
 - End to End CI/CD demonstration

 # Procedures

# Step 1 - Containerizing the application

 - Test the application Locally to confirm is working according to the developer's specification
   - Clone the repository locally
   - Change to the repository directory
   - To run the application locally - (Make sure you have go installed locally)
     - Build the application using the command - go build -o main .
     - Run the application using the command -  ./main
     - Access the application - http://localhost:8080/courses - this ascertain the application is okay to be deployed

 - Write a multistage dockerfile - though we can write a simple docker file but is always good we adhere to best practices
   - In the !st stage - build docker image - download all the dependencies - use any base image - once the application is built
   - In the 2nd stage - you can use a distroless image as the base image - the distroless image will add capabilities such as security and reduced image size - copy the 
     binary built in stage 1 - expose the port - run the application (this is a common practice in the industry)
   - Build the docker image by running the command - docker build -t techkedgec0nnect/profile-go-web-app:v1 .
   - Run the container by using this command - docker run -p 8080:8080 -it techkedgec0nnect/profile-go-web-app:v1
   - Try to access the containerized application to confirm our container is working fine - http://localhost:8080/courses
   - Push the docker image to docker hub repository using the command - docker push techkedgec0nnect/profile-go-web-app:v1

# Step 2 - Writing Kubernetes Manifests
 - Create a new folder name k8s
 - Inside the k8s folder, create another folder called manifests
 - Create deployment.yaml file inside the manifests folder
 - Create service.yaml file inside the manifests folder and ensure that selector matches label in the deployment.yaml file
 - Create ingress.yaml file inside the manifests folder

# Step 3 - Creating EKS Cluster
 - Prerequisites
   - AWS CLI Installation
   - kubectl Installation
   - eksctl Installation
   - Create a programmatic IAM user - Create security credentials and extract those credentials
   - Authenticate your dev/local machine to AWS using the extracted credentials by running the command - aws configure
   - Create relevant policies that would allow crud operations on kubernetes cluster and attach the policy to IAM user
 - Create Kubernetes Cluster
   eksctl create cluster --name tec-web-app-cluster --region eu-west-2 

 - Create a namespace
   kubectl create ns profile-go-web-app

 - Apply the kubernetes manifests files

   - Apply the deployment.yaml file - kubectl apply -f k8s/manifests/deloyment.yaml -n profile-go-web-app
   - Confirm the deployment - kubectl get pods
   - Apply the deployment.yaml file - kubectl apply -f k8s/manifests/service.yaml -n profile-go-web-app
   - Confirm the deployment - kubectl get svc
   - Apply the deployment.yaml file - kubectl apply -f k8s/manifests/ingress.yaml -n profile-go-web-app
   - Confirm the deployment - kubectl get ingress - since the ingress doesn't have an address assigned at the momemnt, we need to edit the svc to nodeport instead of cluster 
     to enable us confirm our application is up and running
   - Edit service.yaml - kubectl edit svc profile-go-web-app
   - To get the application port - kubectl get svc
   - Get node ip address - kubectl get nodes -o wide
   - Access the application using the url - http://node-ip-address:nodeport/courses
