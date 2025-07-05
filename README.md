Jenkins Kubernetes Web Application CI/CD
This project demonstrates a Continuous Integration/Continuous Deployment (CI/CD) pipeline using Jenkins to build a Docker image, push it to Docker Hub, and then deploy a simple web application to a Kubernetes cluster.

Project Overview
The core idea is to automate the process of taking source code (a simple HTML web page), containerizing it with Docker, storing the image, and finally deploying it to a Kubernetes cluster, all orchestrated by Jenkins.

Features
Simple Web Application: A basic index.html served by Nginx.

Dockerization: Dockerfile to create a lightweight Nginx-based Docker image.

Automated Build & Push: Jenkins Pipeline to build the Docker image and push it to Docker Hub.

Kubernetes Deployment: Kubernetes Deployment and Service configurations to run the application in a cluster.

CI/CD Pipeline: Jenkinsfile defining the entire CI/CD workflow, from source code checkout to Kubernetes deployment.

Prerequisites
Before you begin, ensure you have the following installed and configured:

Git: For version control.

Docker: Installed on the machine where Jenkins Agent runs (or Jenkins Master if it's a single node setup) for building and pushing images. Ensure the Jenkins user has permissions to access the Docker daemon (e.g., by being in the docker group).

kubectl: Installed on the Jenkins Agent machine for interacting with the Kubernetes cluster.

Kubernetes Cluster: A running Kubernetes cluster (e.g., Minikube, K3s, or a cloud-managed cluster like GKE, EKS, AKS).

Jenkins: A running Jenkins instance.

If Jenkins is running on the same machine as Kubernetes (but not as a Pod): Ensure the Jenkins user (jenkins) has read access to the Kubernetes kubeconfig file (usually ~/.kube/config). You might need to copy it to /var/lib/jenkins/.kube/config and set appropriate permissions (sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube && sudo chmod 600 /var/lib/jenkins/.kube/config).

If Jenkins is running as a Pod within the Kubernetes cluster: Ensure the Service Account associated with the Jenkins Pod has sufficient RBAC permissions to create/update Deployments and Services in the target namespace.

Docker Hub Account: To store your Docker images.

Project Structure
.
├── Jenkinsfile             # Jenkins Pipeline definition
├── Dockerfile              # Docker image build instructions
├── index.html              # Simple web application HTML file
├── deployment.yaml         # Kubernetes Deployment configuration
└── service.yaml            # Kubernetes Service configuration (NodePort)

Setup Instructions
Follow these steps to set up and run the CI/CD pipeline:

Clone the Repository:

git clone https://github.com/MohammadEsfandiari/K8s-WebApp-with-CICD-
cd K8s-WebApp-with-CICD-

Create Docker Hub Personal Access Token (PAT):

Go to Docker Hub Account Settings.

Navigate to "Security" -> "New Access Token".

Give it a descriptive name (e.g., jenkins-ci-cd).

Grant "Read & Write" permissions for "Repositories".

Copy the generated token immediately. It will only be shown once.

Configure Jenkins Credentials:

Log in to your Jenkins instance.

Go to Dashboard -> Manage Jenkins -> Manage Credentials.

Click on System -> Global credentials (unrestricted).

Click Add Credentials.

Select kind: Username with password.

Username: Your Docker Hub username (e.g., aqx1).

Password: Paste the Docker Hub Personal Access Token (PAT) you generated.

ID: Give it a unique ID (e.g., docker-hub-credentials). Remember this ID.

Click Create.

Update Jenkinsfile Variables:
Open the Jenkinsfile in your local repository and modify the environment section:

environment {
    // DOCKER_REGISTRY: Replace with "docker.io" for Docker Hub.
    // If using another registry (e.g., GCR), put its full address here.
    DOCKER_REGISTRY = "docker.io" // <-- Update this line

    // IMAGE_NAME: Replace "aqx1" with your Docker Hub username.
    // This ensures the image is pushed to your personal repository.
    IMAGE_NAME = "aqx1/my-jenkins-k8s-app" // <-- Update this line (e.g., your_docker_username/my-jenkins-k8s-app)
}

Also, ensure the credentialsId in the Push Docker Image stage matches the ID you set in Jenkins:

withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
    // ...
}

(Replace 'docker-hub-credentials' if you used a different ID).

Commit and Push Changes to Git:

git add .
git commit -m "Configure Jenkinsfile with Docker Hub details"
git push origin main # or master

Create Jenkins Pipeline Job:

In Jenkins, go to Dashboard -> New Item.

Enter an item name (e.g., KubernetesWebApp).

Select Pipeline and click OK.

In the configuration page, scroll down to the Pipeline section.

For Definition, select Pipeline script from SCM.

For SCM, select Git.

Repository URL: Enter your GitHub repository URL (e.g., https://github.com/MohammadEsfandiari/K8s-WebApp-with-CICD-.git).

Credentials: If your repository is private and requires authentication, add appropriate Git credentials here (e.g., SSH key or Username/Password).

Branches to build: */main (or */master if your default branch is master).

Script Path: Enter Jenkinsfile (assuming it's in the root of your repository).

Click Save.

Usage
Run the Jenkins Pipeline:

Go to your newly created Jenkins job.

Click Build Now on the left sidebar.

Monitor the build progress in the Console Output.

Accessing the Deployed Application:
Once the pipeline completes successfully (assuming you temporarily commented out the Cleanup stage in Jenkinsfile for testing), your application will be running in Kubernetes.

Find the NodePort:

kubectl get svc jenkins-k8s-app-service

Look for the NodePort in the PORTS column (e.g., 80:30000/TCP). The NodePort will be a number between 30000-32767.

Find a Cluster Node IP:

kubectl get nodes -o wide

Get the INTERNAL-IP or EXTERNAL-IP of any of your Kubernetes nodes.

Open in Browser:
Navigate to http://<YOUR_NODE_IP>:<YOUR_NODEPORT> in your web browser.
Example: http://192.168.1.100:30000

Troubleshooting Common Issues
ERROR: permission denied while trying to connect to the Docker daemon socket:

The Jenkins user does not have permission to run Docker commands.

Solution: Add the Jenkins user to the docker group on the Jenkins server/agent:
sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins (or restart Jenkins from UI)

ERROR: failed to build: failed to read dockerfile: open Dockerfile: no such file or directory (or for Jenkinsfile, deployment.yaml, service.yaml):

The specified file is not found in the root of your Git repository, or its name is incorrect.

Solution: Ensure Dockerfile, Jenkinsfile, deployment.yaml, and service.yaml are located directly in the root of your Git repository and have the exact correct capitalization. Commit and push any changes.

Error response from daemon: Get "https://registry-1.docker.io/v2/": unauthorized: incorrect username or password:

The Docker Hub credentials (username/password or PAT) configured in Jenkins are incorrect or expired.

Solution: Double-check your Docker Hub username and the Personal Access Token (PAT) in Jenkins Credentials. Ensure the PAT has "Read & Write" access to repositories.

The push refers to repository [docker.io/library/my-jenkins-k8s-app] denied: requested access to the resource is denied:

You are trying to push to an official Docker Hub repository (library/) which is not allowed.

Solution: Ensure your IMAGE_NAME in Jenkinsfile includes your Docker Hub username (e.g., IMAGE_NAME = "your_docker_username/my-jenkins-k8s-app").

error: error validating "deployment.yaml": ... Authentication required (or similar kubectl authentication errors):

kubectl on the Jenkins Agent cannot authenticate with your Kubernetes cluster.

Solution:

If Jenkins is on the same machine as Kubernetes (not as a Pod): Copy your kubeconfig file to /var/lib/jenkins/.kube/config and set correct ownership/permissions (sudo chown -R jenkins:jenkins /var/lib/jenkins/.kube && sudo chmod 600 /var/lib/jenkins/.kube/config).

If Jenkins is a Pod in Kubernetes: Ensure the Jenkins Service Account has the necessary RBAC permissions (e.g., create, get, update, delete for deployments and services) in the target namespace.

Cleanup (Optional)
To remove the deployed application from your Kubernetes cluster, you can run the cleanup commands manually from your server or enable the Cleanup Kubernetes Resources stage in your Jenkinsfile and run the pipeline.

kubectl delete -f service.yaml
kubectl delete -f deployment.yaml

Feel free to contribute to this project or suggest improvements!
