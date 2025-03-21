#  CI/CD PIPELINE WITH AWS EKS AND ECR

Kubernetes Cluster on AWS with Jenkins & ECR

## Overview

This project sets up a Kubernetes cluster on AWS (EKS), with container images deployed in AWS Elastic Container Registry (ECR), and automates deployment using Jenkins. Kubernetes is configured to pull images securely from ECR using a secret.

⸻

## Tools Used
1. Amazon Web Services (AWS) – Cloud provider hosting the Kubernetes cluster.
2. Amazon Elastic Kubernetes Service (EKS) – Managed Kubernetes cluster on AWS.
3. Amazon Elastic Container Registry (ECR) – Stores Docker images.
4. kubectl – Command-line tool to interact with Kubernetes.
5. Jenkins – CI/CD automation server for automating builds and deployments.
6. Docker – Used to containerize the application before deployment.


# Step 1: Create a Kubernetes Cluster on AWS (EKS)

Using eksctl:
``` bash
eksctl create cluster --name my-eks-cluster --region eu-north-1 
```

Verify that the cluster is running:

``` bash
kubectl get nodes
```

# Step 2: Create an AWS ECR Repository

Create a new ECR repository on AWS manually or on the command line:
``` bash
aws ecr create-repository --repository-name my-app --region eu-north-1
```

Authenticate Docker with ECR:
``` bash
aws ecr get-login-password --region eu-north-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.eu-north-1.amazonaws.com
```

# Step 3: Build and Push Docker Image to ECR

1. Build the Docker image:
``` bash
docker build -t <AWS_ACCOUNT_ID>.dkr.ecr.eu-north-1.amazonaws.com/my-app:<version-number> .
```

2. Tag the Docker image:
``` bash
docker tag my-app:<version-number> <AWS_ACCOUNT_ID>.dkr.ecr.eu-north-1.amazonaws.com/my-app:<version-number>
```

3. Push the Docker image to AWS ECR:
``` bash
docker push <AWS_ACCOUNT_ID>.dkr.ecr.eu-north-1.amazonaws.com/my-app:<version-number>
```

# Step 4: Create an Image Pull Secret in Kubernetes

Since AWS ECR is a private repository, Kubernetes needs a secret to pull images. Run the following command:
``` bash
kubectl create secret docker-registry aws-registry-key \
--docker-server=<AWS_ACCOUNT_ID>.dkr.ecr.eu-north-1.amazonaws.com \
--docker-username=AWS \
--docker-password=$(aws ecr get-login-password --region eu-north-1) \
--namespace default
```

Verify that the secret was created:
``` bash
kubectl get secrets
```

# Step 5: Create Kubernetes Deployment File (kubernetes/deployment.yaml)

Modify the deployment file to:
• Use ECR image
• Ensure the image is always pulled (imagePullPolicy: Always)
• Reference the ecr-secret for authentication

Apply the deployment:
``` bash
kubectl apply -f kubernetes/deployment.yaml
```

# Step 6: Create a Kubernetes Service (kubernetes/service.yaml)

Apply the service:
``` bash
kubectl apply -f kubernetes/service.yaml
```

# Step 7: Configure Jenkins container 

1. Install  kubectl command line tool inside Jenkins container.
2. Install aws-iam-authenticator tool inside Jenkins container.
3. Create kubeconfig file to connect to EKS cluster.
4. Add AWS credentials on Jenkins for AWS account authentication.

# Step 8: Automate Deployment with Jenkins

Take a look at the Jenkins file from the project folder.


# Step 9: Verify Deployment

Check deployed pods:
``` bash
kubectl get pods
```
Get service details:
``` bash
kubectl get svc my-app-service
```

Find the EXTERNAL-IP of the service and access the application in a browser.

⸻

## Conclusion

This setup ensures a fully automated deployment pipeline where:
✅ The application is containerized and stored in AWS ECR.
✅ Jenkins automates the build and deployment process.
✅ AWS EKS runs the Kubernetes cluster.
✅ Kubernetes pulls the image from ECR using a secret and updates the deployment automatically.

This guarantees a robust, scalable, and secure CI/CD workflow for Kubernetes on AWS. 🚀
