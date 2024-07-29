# Implementation of an End to End DevOps on a Golang web application. We will implement the following things 

 - Containerization (Multi Stage Docker Build)
 - Creating Kubernetes Manifests
 - Continuous Integration using GitHub Actions
 - Continuous Delivery using Argo CD
 - Kubernetes Cluster creation and setup
 - Helm chart creation and configuration for multiple environments
 - Ingress controller creation, configuration to expose application
 - DNS mapping for our domain
 - End to End CI/CD demonstration

 # Procedures

# Step 1 - Containerizing the application

 - Test the application Locally to confirm is working according to the developer's specification
   - Clone the repository locally
   - Change to the repository directory
   - To run the application locally
     - Build the application using the command - go build -o main .
     - Run the application using the command -  ./main
     - Access the application - http://localhost:8080/courses - this ascertain the application is okay to be deployed

 - Write a multistage dockerfile - though we can write a simple docker file but is always good we adhere to best practices
   - In the !st stage - build docker image - download all the dependencies - use any base image - once the application is built
   - In the 2nd stage - you can use a distroless image as the base image - the distroless image will add capabilities such as security and reduced image size - copy the binary built in stage 1 - expose the port - run the application (this is a common practice in the industry)

# 1st Stage - Build the Go Application
FROM golang:1.21 as base
WORKDIR /app
COPY go.mod .
RUN go mod download
COPY . .
RUN go build -o main .
# EXPOSE 8080
# CMD [*./main]

# Final Stage - With Distroless Image
FROM gcr.io/distroless/base
COPY --from=base /app/main .
COPY --from=base /app/static ./static
EXPOSE 8080
CMD [*./main]

   - Build the docker image by running the command - docker build -t techkedgec0nnect/profile-go-app.web:v1 .
   - Run the container by using this command - docker run -p 8080:8080 -it techkedgec0nnect/profile-go-app-web:v1
   - Try to access the containerized application to confirm our container is working fine - http://localhost:8080/courses
   - Push the docker image to docker hub repository using the command - docker push techkedgec0nnect/profile-go-app.web:v1


