pipeline {
    agent any

    environment {
        IMAGE_NAME = "nikitaram2799/devops-app"
        AWS_REGION = "us-east-2"
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
                    def GIT_SHA = sh(returnStdout: true, script: "git rev-parse --short HEAD").trim()
                    env.IMAGE_TAG = "${IMAGE_NAME}:${GIT_SHA}"
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

        stage('Configure AWS') {
            steps {
                sh '''
                aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                aws configure set region ${AWS_REGION}
                '''
            }
        }

        stage('Update kubeconfig') {
            steps {
                sh "aws eks update-kubeconfig --name devops-cluster --region ${AWS_REGION}"
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh """
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                kubectl set image deployment/devops-app devops-app=${IMAGE_TAG}
                kubectl rollout status deployment/devops-app
                """
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "Deployment failed. Check logs."
        }
    }
}