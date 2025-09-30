pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = "028196693486"          // AWS Account ID
        AWS_REGION = "ap-south-1"                // AWS Region
        IMAGE_REPO = "react-demo"                // ECR Repository name
        ECR_URL = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${IMAGE_REPO}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/Jagannath-bite/reactapp-kubernertes.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build --pull -t ${ECR_URL}:latest ."
            }
        }

        stage('Login to ECR') {
            steps {
                sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
            }
        }

        stage('Push to ECR') {
            steps {
                sh "docker push ${ECR_URL}:latest"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Make sure kubeconfig is available on Jenkins node
                sh "kubectl apply -f k8s-manifests/"
                sh "kubectl rollout status deployment/<your-deployment-name>"
            }
        }
    }
}
