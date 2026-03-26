pipeline {
    agent any

    environment {
        IMAGE_NAME = "nikitaram2799/devops-app"
        AWS_REGION = "us-east-2"                  // Change to your AWS region
        KUBECONFIG_PATH = "/var/jenkins_home/.kube/config"
        DOCKER_CRED_ID = "dockerhub-cred"
        AWS_ACCESS_KEY_ID = credentials('aws-access-key')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-key')
    }

    stages {

        stage('Clone Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nikitaram882799/devops-project.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Get the current Git commit SHA
                    def GIT_SHA = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    env.IMAGE_TAG = "${IMAGE_NAME}:${GIT_SHA}"
                    
                    // Build Docker image with SHA tag
                    sh "docker build -t ${env.IMAGE_TAG} app/"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withDockerRegistry([credentialsId: "${DOCKER_CRED_ID}", url: '']) {
                    sh "docker push ${env.IMAGE_TAG}"
                }
            }
        }

        stage('Configure AWS & Kubeconfig') {
            steps {
                // Update kubeconfig for your EKS cluster
                sh "aws eks update-kubeconfig --name devops-cluster --region ${AWS_REGION}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                // Set new image for deployment
                sh "KUBECONFIG=${KUBECONFIG_PATH} kubectl set image deployment/devops-app devops-app=${env.IMAGE_TAG} --record"

                // Apply other Kubernetes manifests (services, configmaps, etc.)
                sh "KUBECONFIG=${KUBECONFIG_PATH} kubectl apply -f k8s/service.yaml"

                // Verify deployment rollout
                sh "KUBECONFIG=${KUBECONFIG_PATH} kubectl rollout status deployment/devops-app"
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed. Check the logs above."
        }
    }
}