# EKS Demo - React Application Deployment using Jenkins, Docker and Kubernetes

This project demonstrates how to deploy a simple React + Vite application on **Amazon EKS** using:

- Docker
- Amazon ECR
- Kubernetes
- Amazon EKS
- AWS Load Balancer Controller
- Application Load Balancer
- Jenkins CI/CD

The project is designed for beginners and explains the complete flow from building the application to accessing it through an AWS Load Balancer.

---

# 1. Project Overview

The basic flow of this project is:

Developer
   |
   v
GitHub
   |
   v
Jenkins
   |
   +--> Build Docker Image
   |
   +--> Push Image to Amazon ECR
   |
   v
Amazon EKS
   |
   +--> Kubernetes Deployment
   |
   +--> Pods
   |
   +--> Service
   |
   +--> Ingress
   |
   v
AWS Load Balancer Controller
   |
   v
Application Load Balancer
   |
   v
User

The main idea is:

Jenkins builds and pushes the application image, Kubernetes runs that image, and the AWS Load Balancer Controller exposes the application through an AWS Application Load Balancer.

2. What is Docker?

Docker is used to package an application and everything it needs to run into a container image.

For example, our application is:

React + Vite Application

Docker packages it into:

Docker Image

The image can then run inside Kubernetes.

The flow is:

React Application
       |
       v
Dockerfile
       |
       v
Docker Image
       |
       v
Amazon ECR
       |
       v
Amazon EKS
3. What is Kubernetes?

Kubernetes is a system used to run and manage containers.

Instead of manually starting containers, Kubernetes can:

Start containers
Restart failed containers
Run multiple copies of an application
Distribute traffic
Update applications
Scale applications
Monitor application health

For example, if we want two copies of our application:

Kubernetes
    |
    +---- Pod 1
    |
    +---- Pod 2

If one Pod fails, Kubernetes can create another one.

4. What is Amazon EKS?

EKS stands for:

Elastic Kubernetes Service

Amazon EKS is AWS's managed Kubernetes service.

Instead of installing and managing the Kubernetes control plane ourselves, AWS manages it for us.

A simple EKS architecture is:

                 Amazon EKS
                     |
          +----------+----------+
          |                     |
          v                     v
   Control Plane            Worker Nodes
    Managed by AWS               |
                                 |
                       +---------+---------+
                       |                   |
                       v                   v
                     Pod 1               Pod 2
                       |                   |
                       +---------+---------+
                                 |
                                 v
                            Application
5. Important Kubernetes Terms

Before understanding the project, it is important to understand these basic Kubernetes terms.

Term	Simple Meaning
Cluster	Complete Kubernetes environment
Node	Machine that runs Pods
Pod	Smallest unit in Kubernetes that runs a container
Container	Running application inside a Pod
Deployment	Manages Pods
Replica	Number of copies of a Pod
Label	A tag attached to a Kubernetes resource
Selector	Used to find resources using labels
Namespace	Logical separation inside a cluster
Service	Provides stable networking to Pods
Ingress	Defines how external HTTP traffic reaches a Service
Controller	Watches Kubernetes resources and performs actions
ECR	AWS service used to store Docker images
EKS	AWS managed Kubernetes service
ALB	AWS Application Load Balancer
6. Project Structure

The important files in this project are:

eks-demo/
│
├── src/
│   └── React application files
│
├── public/
│
├── Dockerfile
├── Jenkinsfile
├── k8s.yaml
├── iam_policy.json
│
├── package.json
├── package-lock.json
├── vite.config.js
└── index.html
Dockerfile

Used to create the Docker image.

Jenkinsfile

Contains the Jenkins CI/CD pipeline.

k8s.yaml

Contains the Kubernetes configuration.

It creates:

Namespace
Deployment
Service
Ingress
iam_policy.json

Contains AWS permissions required by the AWS Load Balancer Controller.

7. Dockerfile

The Dockerfile uses a multi-stage Docker build.

The basic structure is:

Stage 1
Node.js
   |
   v
Build React Application
   |
   v
dist/

Stage 2
Nginx
   |
   v
Copy dist/
   |
   v
Serve React Application
8. Docker Build Stage

The first stage uses Node.js.

FROM node:24-alpine AS builder

This creates a temporary container with Node.js.

Working Directory
WORKDIR /app

This creates /app as the working directory inside the container.

Copy Package Files
COPY package*.json ./

This copies:

package.json
package-lock.json

into the container.

Install Dependencies
RUN npm ci

This installs the dependencies required by the React project.

Copy Project Files
COPY . .

This copies the application source code into the container.

Build React Application
RUN npm run build

Vite builds the React application.

The production files are generated inside:

dist/
9. Nginx Stage

The second stage uses Nginx.

FROM nginx:1.29-alpine

Nginx is used to serve the production React files.

Copy React Build
COPY --from=builder /app/dist /usr/share/nginx/html

This copies the dist folder generated in the first stage into the Nginx web directory.

Port 80
EXPOSE 80

Nginx listens on port 80.

Start Nginx
CMD ["nginx", "-g", "daemon off;"]

This starts Nginx.

10. Why Use a Multi-Stage Docker Build?

The first stage is only used to build the application.

The final image only contains what is needed to serve the application.

Node.js
   |
   | Build
   v
React dist/
   |
   v
Nginx
   |
   v
Production Container

This keeps the final image cleaner and avoids including unnecessary development tools.

11. Kubernetes YAML

The Kubernetes configuration is stored in:

k8s.yaml

It contains four main resources:

Namespace
    |
    v
Deployment
    |
    v
Pods
    |
    v
Service
    |
    v
Ingress
12. Namespace

The first resource is the Namespace.

apiVersion: v1
kind: Namespace

metadata:
  name: development

A Namespace is like a logical folder inside Kubernetes.

For example:

Kubernetes Cluster
│
├── development
│
├── testing
│
└── production

In this project, our application is deployed inside:

development

Therefore most commands use:

-n development
13. Deployment

The next resource is the Deployment.

apiVersion: apps/v1
kind: Deployment

A Deployment manages Pods.

In this project the Deployment is called:

frontend

and belongs to:

development

The structure is:

Deployment
     |
     +---- Pod
     |
     +---- Pod
14. What is a Replica?

The Deployment contains:

spec:
  replicas: 2

This means:

Kubernetes should maintain two copies of the application.

So Kubernetes creates:

Deployment
    |
    +---- Pod 1
    |
    +---- Pod 2

If one Pod fails:

Pod 1 → Running
Pod 2 → Failed

Kubernetes tries to create another Pod:

Pod 1 → Running
Pod 2 → Failed
Pod 3 → Created

The desired number is again:

2 Pods

Replicas are useful for:

High availability
Handling traffic
Recovering from Pod failures
15. Labels

Labels are tags attached to Kubernetes resources.

The Pods use:

labels:
  app: frontend

This means:

app = frontend

Think of it as a label:

Pod 1
app=frontend

Pod 2
app=frontend
16. Selectors

The Deployment contains:

selector:
  matchLabels:
    app: frontend

This tells the Deployment:

Manage Pods that have the label app=frontend.

The Service also uses:

selector:
  app: frontend

This tells the Service:

Send traffic to Pods that have the label app=frontend.

Therefore labels connect the resources.

Deployment
     |
     | creates
     v
Pods
     |
     | label: app=frontend
     v
Service
     |
     | selector: app=frontend
     v
Pods
17. Understanding spec

You will see spec many times in Kubernetes YAML.

spec describes the desired configuration.

For example:

spec:
  replicas: 2

means:

I want two copies of the application.

For a Pod:

spec:
  containers:

means:

This describes the containers that should run inside the Pod.

18. Pod Template

Inside the Deployment we have:

template:
  metadata:
    labels:
      app: frontend

  spec:
    containers:

The template describes what each Pod should look like.

The Deployment uses this template to create Pods.

Deployment
     |
     | uses template
     v
Pod
     |
     v
Container
19. Containers

The Pod contains:

containers:
  - name: frontend
    image: <ECR_IMAGE>

This tells Kubernetes:

Create a container named frontend using the specified Docker image.

The image is stored in Amazon ECR.

The flow is:

Docker Image
     |
     v
Amazon ECR
     |
     v
EKS Node
     |
     v
Pod
     |
     v
Container
20. Container Port

The container exposes:

ports:
  - containerPort: 80

Our Nginx server listens on port 80.

Therefore:

Pod
 |
 +---- Container
          |
          +---- Nginx
                  |
                  +---- Port 80
21. Resource Requests and Limits

The Deployment defines resources for the container.

Example:

resources:
  requests:
    cpu: "100m"
    memory: "128Mi"

  limits:
    cpu: "500m"
    memory: "512Mi"
Requests

Requests tell Kubernetes the minimum resources needed.

CPU: 100m
Memory: 128Mi
Limits

Limits define the maximum resources the container can use.

CPU: 500m
Memory: 512Mi

This helps Kubernetes schedule Pods properly and prevents a container from consuming unlimited resources.

22. Readiness Probe

The application has a readiness probe.

readinessProbe:
  httpGet:
    path: /
    port: 80

A readiness probe asks:

Is this Pod ready to receive traffic?

If the Pod is not ready, Kubernetes does not send normal Service traffic to it.

Simple meaning:

Readiness Probe
       |
       v
"Is the application ready?"
       |
    +--+--+
    |     |
   Yes    No
    |     |
Traffic  No Traffic
23. Liveness Probe

The application also has a liveness probe.

livenessProbe:
  httpGet:
    path: /
    port: 80

A liveness probe asks:

Is the application still alive and healthy?

If the application becomes unhealthy, Kubernetes can restart the container.

The difference is:

Readiness Probe
    |
    +--> Can this Pod receive traffic?

Liveness Probe
    |
    +--> Is this Pod still alive?
24. Kubernetes Service

The next resource is:

kind: Service

The Service provides stable networking for Pods.

Pods are temporary.

A Pod can be deleted and recreated, which can result in a different IP address.

Therefore, users and other Kubernetes resources should not directly depend on Pod IP addresses.

Instead:

Service
   |
   +---- Pod 1
   |
   +---- Pod 2

The Service provides a stable endpoint.

25. Service Selector

The Service contains:

selector:
  app: frontend

This means:

Find Pods with the label app=frontend.

Our Pods have:

labels:
  app: frontend

Therefore the Service knows which Pods should receive traffic.

26. Service Port

The Service contains:

ports:
  - port: 80
    targetPort: 80

There are two important ports.

port

The port exposed by the Service.

Service → Port 80
targetPort

The port where the application is running inside the Pod.

Pod → Port 80

So:

Service Port 80
       |
       v
Target Port 80
       |
       v
Container Port 80
27. Why is the Service ClusterIP?

The Service uses:

type: ClusterIP

ClusterIP means:

The Service is available inside the Kubernetes cluster.

It is not directly exposed to the public internet.

This project uses an Ingress and AWS Application Load Balancer for external access.

The architecture is:

Internet
   |
   v
Application Load Balancer
   |
   v
Ingress
   |
   v
ClusterIP Service
   |
   v
Pods
28. What is Ingress?

Ingress is a Kubernetes resource used to define external HTTP/HTTPS access to applications.

It defines rules such as:

/

should go to:

frontend-service

The flow is:

Internet
   |
   v
Load Balancer
   |
   v
Ingress
   |
   v
Service
   |
   v
Pods

Important:

Ingress itself does not create an AWS Application Load Balancer.

A controller is required to watch the Ingress and create the AWS resources.

29. Ingress Class

The project uses:

ingressClassName: alb

This tells Kubernetes that the Ingress should be handled by the AWS Load Balancer Controller.

30. Ingress Annotations

The Ingress contains AWS-specific annotations.

For example:

alb.ingress.kubernetes.io/scheme: internet-facing

This tells the AWS Load Balancer Controller to create a public-facing load balancer.

Another annotation is:

alb.ingress.kubernetes.io/target-type: ip

This tells the controller to use Pod IP addresses as targets.

The project also configures:

HTTP port 80
Health check path /
31. Complete Kubernetes Flow

The complete Kubernetes architecture is:

                 Kubernetes Cluster
                        |
                        v
                  development
                   Namespace
                        |
                        v
                   Deployment
                   frontend
                        |
                 replicas: 2
                        |
             +----------+----------+
             |                     |
             v                     v
           Pod 1                 Pod 2
       app=frontend          app=frontend
             |                     |
             +----------+----------+
                        |
                        v
                    Service
                frontend-service
                   ClusterIP
                        |
                        v
                    Ingress
               frontend-ingress
                        |
                        v
             AWS Load Balancer
32. What is the AWS Load Balancer Controller?

The AWS Load Balancer Controller is a Kubernetes controller that connects Kubernetes with AWS load-balancing services.

It watches Kubernetes resources such as:

Ingress
Service

and creates or manages AWS load-balancing resources.

The basic idea is:

Kubernetes Ingress
        |
        v
AWS Load Balancer Controller
        |
        v
AWS APIs
        |
        v
Application Load Balancer
33. Why Do We Need the Controller?

This is an important concept.

Kubernetes is designed to work with many platforms:

AWS
Azure
Google Cloud
On-Premises

Kubernetes itself does not contain all the AWS-specific logic needed to create and configure an AWS Application Load Balancer.

The AWS Load Balancer Controller provides that AWS-specific functionality.

For example, when we create:

kind: Ingress

spec:
  ingressClassName: alb

the controller notices the Ingress.

It then communicates with AWS and creates/configures the required resources.

Conceptually:

You create Ingress
        |
        v
Controller notices Ingress
        |
        v
Controller calls AWS APIs
        |
        v
AWS creates/configures ALB
        |
        v
ALB sends traffic to application
34. What Does a Controller Do?

A Kubernetes controller continuously watches the cluster.

It compares:

Desired State

with:

Actual State

For example:

Desired:

Ingress should have an ALB

The controller checks AWS.

If the ALB does not exist:

Create ALB

If something needs to change:

Update ALB

Therefore:

Desired State
      |
      v
Controller
      |
      v
Actual AWS Resources

This is why the controller is important.

35. IAM Permissions for the Controller

The AWS Load Balancer Controller needs permission to call AWS APIs.

The project contains:

iam_policy.json

This policy provides the permissions required by the controller to manage AWS load-balancing resources.

The basic relationship is:

AWS Load Balancer Controller
             |
             v
       IAM Role/Policy
             |
             v
          AWS APIs
             |
             v
       AWS Resources
36. Create an EKS Cluster

There are multiple ways to create an EKS cluster.

For beginners, eksctl is one of the easiest methods.

First install:

AWS CLI
kubectl
eksctl
Helm
Docker

Check the installations:

aws --version
kubectl version --client
eksctl version
helm version
docker --version
37. Configure AWS CLI

Run:

aws configure

Enter your AWS credentials.

Set the region to:

ap-south-1

Verify your AWS identity:

aws sts get-caller-identity

If the command returns your AWS account information, the AWS CLI is configured correctly.

38. Create the EKS Cluster

Create the cluster using:

eksctl create cluster \
  --name demo-cluster \
  --region ap-south-1

This creates the EKS cluster and the required worker nodes.

Cluster creation can take several minutes.

39. Verify the EKS Cluster

Check the cluster:

eksctl get cluster

Check the worker nodes:

kubectl get nodes

You should see nodes with:

STATUS
Ready

For example:

NAME                         STATUS
ip-10-0-1-xxx.ec2.internal   Ready
ip-10-0-2-xxx.ec2.internal   Ready
40. Configure kubectl

Run:

aws eks update-kubeconfig \
  --region ap-south-1 \
  --name demo-cluster

This configures kubectl to communicate with the EKS cluster.

Verify:

kubectl get nodes
41. Install AWS Load Balancer Controller

Before applying the Ingress, the AWS Load Balancer Controller should be installed.

The installation has three main parts:

1. IAM Policy
       |
       v
2. IAM Service Account
       |
       v
3. Helm Installation
42. Create IAM Policy

The project contains:

iam_policy.json

Create the IAM policy:

aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json

Replace the policy with the current AWS-recommended policy if the controller version changes.

43. Create IAM Service Account

Create the service account:

eksctl create iamserviceaccount \
  --cluster=demo-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --attach-policy-arn=arn:aws:iam::<AWS_ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy \
  --override-existing-serviceaccounts \
  --region ap-south-1 \
  --approve

Replace:

<AWS_ACCOUNT_ID>

with your AWS account ID.

The service account connects Kubernetes with AWS IAM permissions.

Kubernetes Service Account
            |
            v
         IAM Role
            |
            v
      IAM Permissions
            |
            v
        AWS APIs
44. Install Helm Chart

Add the AWS EKS Helm repository:

helm repo add eks https://aws.github.io/eks-charts

Update the repository:

helm repo update

Install the controller:

helm install aws-load-balancer-controller \
  eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=demo-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller

The important options are:

clusterName

Tells the controller which EKS cluster it belongs to.

serviceAccount.create=false

Means Helm should not create another service account.

serviceAccount.name

Tells Helm to use the service account we already created.

45. Verify AWS Load Balancer Controller

Check the deployment:

kubectl get deployment \
  aws-load-balancer-controller \
  -n kube-system

Check the Pods:

kubectl get pods -n kube-system

The controller Pod should show:

Running

You can also check its logs:

kubectl logs \
  deployment/aws-load-balancer-controller \
  -n kube-system
46. Deploy the Kubernetes Application

Now apply the Kubernetes configuration:

kubectl apply -f k8s.yaml

This creates:

Namespace
Deployment
Service
Ingress
47. Check the Namespace

Run:

kubectl get namespaces

You should see:

development
48. Check the Pods

Run:

kubectl get pods -n development

Because the Deployment has:

replicas: 2

you should see two Pods.

Example:

NAME                        READY   STATUS
frontend-xxxxx              1/1     Running
frontend-yyyyy              1/1     Running
49. Check the Deployment

Run:

kubectl get deployment -n development

You should see:

NAME       READY
frontend   2/2
50. Check the Service

Run:

kubectl get service -n development

You should see:

frontend-service

The Service is a:

ClusterIP

so it is available internally inside the Kubernetes cluster.

51. Check the Ingress

Run:

kubectl get ingress -n development

Initially, the address may be empty.

Wait for the AWS Load Balancer Controller to create the Application Load Balancer.

Run the command again:

kubectl get ingress -n development

Eventually, an address similar to this will appear:

xxxx.ap-south-1.elb.amazonaws.com

Open this address in your browser.

The React application should appear.

52. Complete Application Traffic Flow

When a user opens the website:

                User Browser
                     |
                     v
          Application Load Balancer
                     |
                     v
                  Ingress
                     |
                     v
            frontend-service
                     |
              +------+------+
              |             |
              v             v
            Pod 1         Pod 2
              |             |
              v             v
             Nginx        Nginx
              |             |
              +------+------+
                     |
                     v
              React Application
53. Jenkins CI/CD

The Jenkinsfile automates the deployment process.

The pipeline follows these stages:

Checkout
   |
   v
Login to Amazon ECR
   |
   v
Build Docker Image
   |
   v
Tag Docker Image
   |
   v
Push Docker Image
   |
   v
Configure kubectl
   |
   v
Deploy to EKS
   |
   v
Verify Rollout
   |
   v
Verify Deployment
   |
   v
Post Actions
54. Jenkins Environment

The Jenkinsfile defines environment variables such as:

environment {
    AWS_REGION = 'ap-south-1'
    ACCOUNT_ID = '<AWS_ACCOUNT_ID>'
    REPOSITORY = 'frontend'
    IMAGE_TAG = "${BUILD_NUMBER}"
    ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${REPOSITORY}"
    EKS_CLUSTER = 'demo-cluster'
    SERVICE_NAME = 'frontend'
    K8S_NAMESPACE = 'development'
}

Environment variables make the pipeline easier to maintain.

For example:

AWS_REGION

stores:

ap-south-1

and:

EKS_CLUSTER

stores:

demo-cluster
55. Jenkins Build Number

The image tag is based on:

IMAGE_TAG = "${BUILD_NUMBER}"

Suppose Jenkins runs build number:

15

The Docker image becomes:

frontend:15

This allows each Jenkins build to have its own image version.

For example:

Build 10 → frontend:10
Build 11 → frontend:11
Build 12 → frontend:12

This makes it easier to identify which version was deployed.

56. Stage 1 - Login to Amazon ECR

Jenkins first logs in to Amazon ECR.

The command is:

aws ecr get-login-password \
  --region ${AWS_REGION} |
docker login \
  --username AWS \
  --password-stdin \
  ${ECR_URI}

This allows Docker to push images to the ECR repository.

The flow is:

Jenkins
   |
   v
AWS CLI
   |
   v
Amazon ECR
   |
   v
Docker Login
57. Stage 2 - Build Docker Image

Jenkins builds the Docker image:

docker build -t ${REPOSITORY}:${IMAGE_TAG} .

For example:

docker build -t frontend:15 .

Docker reads:

Dockerfile

and creates:

frontend:15
58. Stage 3 - Tag Docker Image

Jenkins tags the image with the ECR repository name.

For example:

docker tag frontend:15 \
  <AWS_ACCOUNT_ID>.dkr.ecr.ap-south-1.amazonaws.com/frontend:15

The image now has the ECR name.

Conceptually:

frontend:15
     |
     v
ECR_URI:15

The pipeline also tags the image as:

latest
59. Stage 4 - Push Docker Image

Jenkins pushes the image to Amazon ECR:

docker push ${ECR_URI}:${IMAGE_TAG}

It also pushes the latest tag.

The flow is:

Jenkins
   |
   v
Docker Image
   |
   v
Amazon ECR

ECR now stores the image that Kubernetes will use.

60. Stage 5 - Configure kubectl

Jenkins needs access to the EKS cluster.

It runs:

aws eks update-kubeconfig \
  --region ${AWS_REGION} \
  --name ${EKS_CLUSTER}

This configures kubectl.

The flow becomes:

Jenkins
   |
   | kubectl
   v
Amazon EKS

The Jenkins machine must have:

AWS CLI
kubectl
AWS credentials
Permission to access the EKS cluster
61. Stage 6 - Deploy to EKS

Jenkins updates the Deployment with the newly built image.

The command is:

kubectl set image deployment/${SERVICE_NAME} \
  ${SERVICE_NAME}=${ECR_URI}:${IMAGE_TAG} \
  -n ${K8S_NAMESPACE}

For example:

frontend:14

may be replaced with:

frontend:15

Kubernetes then starts a rolling update.

62. What is a Rolling Update?

Suppose the current Pods use:

frontend:14

Jenkins deploys:

frontend:15

Kubernetes gradually replaces the old Pods.

Conceptually:

Old Version
frontend:14
frontend:14

       |
       v

New Version Created
frontend:15

       |
       v

Old Version Removed

       |
       v

frontend:15
frontend:15

This avoids manually stopping the entire application.

63. Stage 7 - Verify Rollout

Jenkins runs:

kubectl rollout status deployment/frontend \
  -n development

This waits for the Deployment to successfully roll out.

If successful, Jenkins continues.

If the rollout fails, Jenkins marks the pipeline as failed.

64. Stage 8 - Verify Deployment

Jenkins checks the Kubernetes resources.

For example:

kubectl get pods -n development
kubectl get svc -n development
kubectl get ingress -n development
kubectl get deployment -n development

This gives a basic verification that the application has been deployed.

65. Jenkins Post Section

The Jenkinsfile contains a post section.

The post section runs after the pipeline stages finish.

It handles:

success
failure
always
66. Post - Success

When the pipeline succeeds:

success {
    echo "Deployment Successful"
}

Jenkins prints:

Deployment Successful

This tells us that the pipeline completed successfully.

67. Post - Failure

If the deployment fails, Jenkins attempts to roll back the Deployment.

The command is:

kubectl rollout undo deployment/frontend \
  -n development

This tells Kubernetes to return to the previous Deployment revision.

For example:

Version 14
    |
    v
Version 15
    |
    | Failure
    v
Rollback
    |
    v
Version 14

This helps prevent a failed deployment from remaining active.

68. Why Do We Use || true?

The rollback command may use:

|| true

This means:

Even if the rollback command fails, do not create another failure from that rollback command.

It is used as a defensive shell technique.

69. Post - Always

The always block runs whether the pipeline succeeds or fails.

The Jenkinsfile performs Docker cleanup:

docker image prune -af

and workspace cleanup:

cleanWs()
70. Why Do We Use Docker Image Prune?

Every Jenkins build creates Docker images.

For example:

Build 1 → frontend:1
Build 2 → frontend:2
Build 3 → frontend:3
Build 4 → frontend:4

Unused images can consume disk space on the Jenkins machine.

Therefore:

docker image prune -af

removes unused Docker images from the Jenkins machine.

Important:

This does NOT delete images from Amazon ECR.

It only cleans unused images from the Jenkins machine.

71. Why Do We Use cleanWs()?

Jenkins creates a workspace for every job.

The workspace contains files such as:

Source code
package files
Dockerfile
Jenkins files

After the build finishes, these files are no longer required by that build.

cleanWs()

cleans the Jenkins workspace.

Therefore:

docker image prune
        +
cleanWs()
        |
        v
Cleaner Jenkins machine
72. Complete Jenkins Flow

The complete CI/CD process is:

Developer
    |
    v
Push Code
    |
    v
GitHub
    |
    v
Jenkins
    |
    v
Checkout Code
    |
    v
Login to ECR
    |
    v
Build Docker Image
    |
    v
Tag Docker Image
    |
    v
Push Image to ECR
    |
    v
Configure kubectl
    |
    v
Connect to EKS
    |
    v
Update Deployment
    |
    v
Rolling Update
    |
    v
Verify Rollout
    |
    v
Verify Deployment
    |
    +-------------------+
    |                   |
    v                   v
 SUCCESS              FAILURE
    |                   |
    v                   v
Success Message      Rollback
    |                   |
    +---------+---------+
              |
              v
          Cleanup
73. How Does the Docker Image Reach EKS?

Jenkins does not directly copy the Docker image into the Pods.

The process is:

Jenkins
   |
   | docker push
   v
Amazon ECR
   |
   | Kubernetes tells Pod which image to use
   v
EKS Worker Node
   |
   | pulls image
   v
Pod
   |
   v
Container

Therefore:

Jenkins → ECR → EKS → Pod
74. Complete Project Architecture
                         GitHub
                            |
                            v
                         Jenkins
                            |
              +-------------+-------------+
              |             |             |
              v             v             v
          Docker Build   ECR Login     AWS CLI
              |
              v
         Docker Image
              |
              v
        Amazon ECR
              |
              v
         Amazon EKS
              |
       +------+------+
       |             |
       v             v
 Namespace       Kubernetes
development       Deployment
                       |
                  replicas: 2
                       |
             +---------+---------+
             |                   |
             v                   v
           Pod 1               Pod 2
             |                   |
             +---------+---------+
                       |
                       v
                    Service
                       |
                       v
                    Ingress
                       |
                       v
          AWS Load Balancer Controller
                       |
                       v
          Application Load Balancer
                       |
                       v
                    Internet
                       |
                       v
                      User
75. Useful Kubernetes Commands
Check Nodes
kubectl get nodes
Check Namespaces
kubectl get namespaces
Check Pods
kubectl get pods -n development
Check Deployments
kubectl get deployments -n development
Check Services
kubectl get services -n development
Check Ingress
kubectl get ingress -n development
Check Everything
kubectl get all -n development
76. View Pod Details

If a Pod is not working:

kubectl describe pod <pod-name> \
  -n development

This shows:

Pod status
Events
Container information
Image
Probes
Scheduling information
Errors
77. View Pod Logs

Run:

kubectl logs <pod-name> \
  -n development

Logs are useful for finding application or container problems.

78. Check Deployment Rollout
kubectl rollout status deployment/frontend \
  -n development
79. View Rollout History
kubectl rollout history deployment/frontend \
  -n development
80. Manually Roll Back

If required:

kubectl rollout undo deployment/frontend \
  -n development
81. Troubleshooting
Pods are Pending

Run:

kubectl describe pod <pod-name> \
  -n development

Possible reasons:

Not enough CPU
Not enough memory
Worker node problem
Scheduling problem
ImagePullBackOff

Run:

kubectl describe pod <pod-name> \
  -n development

Possible reasons:

Incorrect ECR image
Image does not exist
Incorrect AWS permissions
EKS node cannot access ECR
Ingress Has No Address

Run:

kubectl get ingress -n development

Then:

kubectl describe ingress frontend-ingress \
  -n development

Check the controller:

kubectl get pods -n kube-system

Make sure:

aws-load-balancer-controller

is running.

Load Balancer Controller Not Running

Check:

kubectl get deployment \
  aws-load-balancer-controller \
  -n kube-system

Check logs:

kubectl logs \
  deployment/aws-load-balancer-controller \
  -n kube-system

Check IAM permissions if the controller cannot create AWS resources.

Jenkins Cannot Access EKS

Check AWS credentials:

aws sts get-caller-identity

Check EKS access:

kubectl get nodes

Jenkins needs:

AWS CLI
kubectl
AWS credentials
EKS permissions
Docker Build Fails

Try building manually:

docker build -t frontend:test .

Check:

Dockerfile
package.json
package-lock.json
