# Implementation of an End to End DevOps on a Golang web application. We will implement the following things 

 - Containerization (Multi Stage Docker Build)
 - Creating Kubernetes Manifests
 - Kubernetes Cluster creation and setup
 - Ingress controller creation, configuration to expose application
   - DNS mapping for our domain
 - Helm chart creation and configuration for multiple environments
 - Continuous Integration using GitHub Actions
 - Continuous Delivery using Argo CD
 - End to End CI/CD demonstration
 - Delete the EKS Kubernetes Cluster

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
   - Confirm the deployment - kubectl get ingress - since the ingress doesn't have an address assigned at the momemnt, we 
     need to edit the svc to nodeport instead of cluster - to enable us confirm our application is up and running
   - Edit service.yaml - kubectl edit svc profile-go-web-app
   - To get the application port - kubectl get svc
   - Get node ip address - kubectl get nodes -o wide
   - Access the application using the url - http://node-ip-address:nodeport/courses

#  Step 4 - Create an Ingress Controller
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.11.1/deploy/static/provider/aws/deploy.yaml
   - Confirm nginx controller deployment - kubectl get pods -n ingress-nginx
   - To confirm that the ingress has an address assigned - kubectl get ingress - you will notice that the ingress now has 
     fqdn address address assigned. This assigned fqdn is the address associated with the network loadbalancer created by 
     the ingress controller. But with this loadbalancer we still can't access our application because in the ingress 
     configuration we have defined that request should only be accepted from profile-go-web-app.local.
   - To make the application accessible - get the IP address of the loadbalancer created by ingress controller and map it to 
     profile-go-web-app.local
   - To get the ip address of the loadbalancer fqdn addrees - nslookup loadbalancer fqdn
   - To map the laodbalancer ip address to profile-go-web-app.local - you need to edit the dev/local machine host file
   - To edit the host file of your dev/local machine - vi /etec/hosts map the loadbalancer ip address to profile-go-web- 
     app.local and save your configuration
     35.178.58.20 profile-go-web-app.local 
     18.171.118.93 profile-go-web-app.local
     13.41.84.222 profile-go-web-app.local 
   - You can now access the application - http://profile-go-web-app.local/courses

# Ingress Controller
  - In Kubernetes, an Ingress Controller is a specialized load balancer responsible for managing access to services within a 
    cluster. 
  - It enables external HTTP and HTTPS traffic to reach these services, providing a way to expose multiple services using a 
    single IP address. 
  - The Ingress Controller operates by monitoring Ingress resources and configuring underlying load balancers accordingly.

  # There are 3 components with respect to ingress
    - Ingress
    - Ingress Controller
    - Loadbalancer

  - Ingress controller monitor the ingress resources and create load balancer in your Kubernetes cluster because you can't 
    create load balancer on your own
  - Ingress Class Name is meant for ingress resource to be identified by ingress controller

# Step 5 - Helm Chart Creation

  - Make sure helm is installed on your local/dev machine
  - Confirm helm installation - helm version
  - Create a folder called helm in your project directory
  - Change to the helm directory and run the command - helm create profile-go-web-app
  - Change directory to the profile-go-web-app-chart directory just created - list the content ls - you will notice that 
    some configuration files have already been created Chart.yaml, charts, templates and values.yaml. 
  - Chart.yaml provides information of the chart such as name, description, chart version, application version, type of 
    chart and can be assumed to be a metadata
  - template directory contains imported configuration templates - it comprises NOTES.txt, deployment.yaml, ingress.yaml 
    serviceaccount.yaml, _helpers.tpl, hpa.yaml service.yaml and test. It is always advisable to remove everything in the 
    template folder (rm -rf *) and replace with all the files inside the manifests directory in this case (deployment.yaml, 
    ingress.yaml and service.yaml) (cp ../../../k8s/manifests/* . - With helm chart we can add variable to our yaml files 
    for instance in the deployment.yaml file we can replace the image tag value of techkedgec0nnect/profile-go-web-app:v1 
    with {{ .Values.image.tag }} - this implies that the helm would look for tag in the Values.yaml file whenever it is 
    executed.
  - Define the tag variable value in values.yaml by editing the content of the values.yaml file generated when created helm 
    chart

replicaCounts: 1

images:
  repositories: techkedgec0nnect/profile-go-web-app
  pullPolicy:  IdNotPresent
  # Overrides the image tag whose default is the chart appversion
  tag: "v1"

  - To verify the our helm is working fine, we need to run the command helm install chart ./profile-go-web-app but before 
    doing this we need to delete existing configuration
    kubectl delete deploy profile-go-web-app
    kubectl delete svc profile-go-web-app-svc
    kubectl delete ing profile-go-web-app-ingress
  - From the helm directory run the command - helm install profile-go-web-app ./profile-go-web-app - This command 
    would deploy our kubernetes resources (deployment, service and ingress) We can verify the resources deployment by 
    running the follwing commands
    kubectl  get deployment -n tec-profile-go-web-app
    kubectl  get svc -n tec-profile-go-web-app
    kubectl  get ingress -n tec-profile-go-web-app
  - To uninstall our resources run the command - helm uninstall profile-go-web-app -n tec-profile-go-web-app

# Helm Chart
A Helm chart is a package manager for Kubernetes, similar to what APT is to Debian or Yum is to RedHat. 
It simplifies the process of defining, installing, and upgrading complex Kubernetes applications. 

- Packaging Kubernetes Applications:
A Helm chart bundles all the Kubernetes resources (like deployments, services, and config maps) necessary to run an application into a single package.
With helm chart we can add variable to our yaml files

# Step 6 - Implement Continuous Integration Using GitHub Action

  - Implement multiple CI stages
    - stage 1 - Build and Test (Unit Test)
    - stage 2 - Run static code analysis
    - stage 3 - Create docker image and push the docker image
    - stage 4 - Update Helm chart values.yaml file with the created docker image tag - After this is done, we move to the 
      Continuous deployment stage where ArgoCD is introduced. The ArgoCD pulled the helm chart after it is updated and 
      deployed it to the Kubernetes cluster
    - To create CI pipeline using GitHub Action - create a .github folder in your project directory and inside the .github 
      folder create another folder called workflows and inside the workflows you can create file with any name but in this 
      case let's call it profile-go-web-app-CI.yaml
    - To create your pipeline for teh workflow inside the profile-go-web-app.CI.yaml file - start by providing a name for 
      the pipeline/workflow - Deploy Profile GO Web Application
    - Provide an action when GitHub has to run this pipeline automatically

# Trigger: The workflow is triggered on every push to the main branch and ensure any update to README.md and helm directory is ignored
on: 
  push:
    branches:
      - main
    paths-ignore:
      - 'README.md'
      - 'helm/**'

  - After defining the workflow trigger, you start writing the jobs - in github action inside the jobs we start writing the 
    stages. All our stages would be defined inside the jobs. You can get all the GitHub Action stage synthax in the GitHub 
    Marketplace by just searching

  - To create docker hub repository username secret in github - go to settings - Click secrets and variables - click actions 
    - click new repository secret - provide secret name (DOCKERHUB_USERNAME) - provide docker hub username value in the 
      secret text box and click add secret.
  - To create docker hub repository token secret in github - go to settings - Click secrets and variables - click actions - 
    click new repository secret - provide secret name (DOCKERHUB_TOKEN) - provide docker hub token value in the secret text 
    box and click add secret. 
  - To generate docker hub token from docker hub - go to account settings - click personal access tokens - click generate 
    new token - give the token description/name - give your access permissions - click generate - copy and save the 
    generated token 
  - To generate github login token - click profile - click settings - click developer settings - click personal access 
    tokens - click tokens classic - click generate new token - click generate new token - name the token - select the scope 
    of the token - click generate token  - copy and save the generated token
  - To create github repository token secret in github - go to settings - Click secrets and variables - click actions - 
    click new repository secret - provide secret name (DOCKERHUB_USERNAME) - provide docker hub username value in the secret 
    text box and click add secret.
  - Commit your code to the repo - git add . - git commit -m"update: CI Implementation of github action workflow" - git push
  - To verify if the github action workflow is running - click action - you will see all workflow running stages as defined 
    in the CI
  - Upon successful completion of the github action workflow confirm that the docker image is pushed to docker hub with a 
    new generated git runner ID tag 
  - To confirm that the image tag is updated in the value.yaml file - go to helm directory - navigate to the value.yaml file 
    to see the image tag value

# Step 7 - Implement Continuous Delivery Using ARGOCD

  - Install Argo CD using manifests
    - kubectl create namespace argocd
    - kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

  - To access the Argo CD UI (Loadbalancer service)
    - kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

  - To get the Loadbalancer service IP
    - kubectl get svc argocd-server -n argocd - get the argocd external address/ip and paste in the browser to access it or 
      you can as well get the argocd external IP address by running the command
    - kubectl get nodes -o wide - copy the external ip address, paste in the browser, add the port number in order to access 
      the argocd UI
    - provide the username - admin
    - for the password you need to run this command - kubectl get secret -n argocd - as part of the output you will find 
      argocd-initial-admin-Secret 
    - to extract the argocd secret run the command - kubectl edit secret argocd-initial-admin-secret -n argocd - extract the 
      base64 encoded password (MVg3cTZRU1FVSFZmREJkag==)
    - to decode the password use this command - echo MVg3cTZRU1FVSFZmREJkag== | base64 --decode
    - provide the decoded password on argocd UI to sign-in - note argocd is on the same kubernetes cluster
    - click new app - enter app name (profile-go-web-app) - select default project - sync policy (select automated) - select 
      self-heal - provide the repository url (argocd will identified the helm chart path automatically) - select defaut for 
      cluster url - provide namespace (make it default) - use the value.yaml provide in github - click create. Argocd will 
      look for all the files in your repository within the helm chart, it will update the values.yaml file with all the 
      changes that are required in ur deployment environment
    - to confirm argocd has deployed our application automatically run the following command
      - kubectl get deploy - confirm our deployment is created
      - kubectl get svc - confirm service is created
      - kubectl get ing - confirm ingress is created

# Step 8 - Verify end to end CICD by making changes in our code

   - Try modify the content of home.html file in the static content directory - Learn DevOps from Basics By Samson Wahab
   - Commit the changes
   - The commit should automatically trigger the CI - buid & test the app - conduct cpde quality analaysis- build a new 
     docker image new git runner ID tag and push to docker hub with - the image tag is updated in the helm chart values.yaml 
     file - confirm the new tag created in the docker hub repository is the same as the updated image tag in helm chart 
     values.yaml file
   - Argocd should pick up the new change and deploy the application latest version - argocd by default look for the new 
     changes
   - To confirm the new changes run the command - kubectl edit deploy - you will notice that the application image has 
     chnaged from the previous version to the newly created image version
   - Verify by the changes by browsing the application url - navigate the the home directory to see the actual changes made.

   - With this we have been able to automate the profile-go-web-app application deployment by implementing an end to end 
     CICD workflow using github action for the CI and Argocd for the CD  

# Step 9 - Delete the Kubernetes Cluster

  - eksctl delete cluster --name tec-web-app-cluster --region eu-west-2




    

