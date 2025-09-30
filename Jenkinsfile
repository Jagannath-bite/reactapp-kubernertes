pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '028196693486'
        AWS_REGION     = 'ap-south-1'
        ECR_REPO       = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/reactapp-k8s"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
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

        stage('Login & Push to ECR') {
            steps {
                // Use AWS credentials stored in Jenkins
                withAWS(credentials: 'awscred', region: "${AWS_REGION}") {
                    script {
                        sh """
                          aws ecr get-login-password --region ${AWS_REGION} \
                            | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                          docker tag reactapp:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                          docker push ${ECR_REPO}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    // Update deployment image and apply Kubernetes manifests
                    sh "kubectl set image deployment/reactapp-deployment reactapp=${ECR_REPO}:${IMAGE_TAG} --record || true"
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
                    sh "kubectl apply -f ingress.yaml"
                }
            }
        }
    }
}
