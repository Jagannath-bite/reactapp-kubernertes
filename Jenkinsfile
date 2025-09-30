pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        ECR_REPO = '028196693486.dkr.ecr.ap-south-1.amazonaws.com/reactapp-k8s'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                     git branch: 'master', url: 'https://github.com/Jagannath-bite/reactapp-kubernertes.git'

            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t reactapp:${IMAGE_TAG} ."
                }
            }
        }

        stage('Login to ECR') {
            steps {
                script {
                    sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    sh "docker tag reactapp:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}"
                    sh "docker push ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl set image deployment/reactapp-deployment reactapp=${ECR_REPO}:${IMAGE_TAG} --record"
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                    sh "kubectl apply -f ingress.yaml"
                }
            }
        }
    }
}
