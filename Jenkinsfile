pipeline {
    agent any

    environment {
       
        DOCKER_REGISTRY = "docker.io" 
        IMAGE_NAME = "my-jenkins-k8s-app"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def imageTag = "${env.DOCKER_REGISTRY}/${env.IMAGE_NAME}:${env.BUILD_NUMBER}"
                    env.DOCKER_IMAGE_FULL_TAG = imageTag 
                    echo "Building Docker image: ${imageTag}"
                    sh "docker build -t ${imageTag} ."
                    echo "Docker image built: ${env.DOCKER_IMAGE_FULL_TAG}"
                }
            }
        }

        
        stage('Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: '87338dd1-35d4-4456-a0a1-2e4ce54dc65b', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        echo "Logging into Docker Registry..."
                        sh "echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin ${DOCKER_REGISTRY}"
                        echo "Pushing Docker image: ${env.DOCKER_IMAGE_FULL_TAG}"
                        sh "docker push ${env.DOCKER_IMAGE_FULL_TAG}"
                        echo "Docker image pushed."
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Updating image tag in deployment.yaml..."
                    sh "sed -i 's|image: my-jenkins-k8s-app:latest|image: ${env.DOCKER_IMAGE_FULL_TAG}|g' deployment.yaml"

                    echo "Applying Kubernetes Deployment and Service..."
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"

                    echo "Waiting for Kubernetes Deployment to be ready..."
                    sh "kubectl rollout status deployment/jenkins-k8s-app-deployment"

                    echo "Application deployed to Kubernetes!"
                    echo "To access the application, find the NodePort for jenkins-k8s-app-service:"
                    echo "kubectl get svc jenkins-k8s-app-service"
                    echo "Then, navigate to http://<your-cluster-node-ip>:<NodePort>"
                }
            }
        }

        stage('Test Application (Optional)') {
            steps {
                script {
                    echo "You can add your automated tests here, e.g., using curl against the NodePort."
                    echo "This requires knowing the Node IP and NodePort, which can be retrieved dynamically."
                    
                }
            }
        }

        stage('Cleanup Kubernetes Resources (Optional)') {
            steps {
                script {
                    echo "Deleting Kubernetes Deployment and Service..."
                    sh "kubectl delete -f service.yaml || true"
                    sh "kubectl delete -f deployment.yaml || true"
                    echo "Kubernetes resources cleaned up."
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished."
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

