pipeline {
    agent any

    environment {
        AWS_ACCOUNT_ID = '028196693486'
        AWS_REGION     = 'ap-south-1'
        CLUSTER_NAME   = 'react-cluster'
        ECR_REPO_NAME  = 'reactapp'
        ECR_REPO       = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}"
        IMAGE_TAG      = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code..."
                git branch: 'master', url: 'https://github.com/Jagannath-bite/reactapp-kubernertes.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh """
                        set -x
                        docker build -t ${ECR_REPO}:${IMAGE_TAG} .
                    """
                }
            }
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
        }

        stage('Login & Push to ECR') {
            steps {
                script {
                    echo "Logging into ECR and pushing image..."
                    withAWS(credentials: 'awscred', region: "${AWS_REGION}") {
                        sh """
                            set -x
                            aws ecr get-login-password --region ${AWS_REGION} \
                              | docker login --username AWS --password-stdin ${ECR_REPO}

                            docker tag ${ECR_REPO}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}
                            docker push ${ECR_REPO}:${IMAGE_TAG}
                        """
                    }
                }
            }
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
        }

        stage('Configure kubectl') {
            steps {
                script {
                    echo "Configuring kubectl..."
                    withAWS(credentials: 'awscred', region: "${AWS_REGION}") {
                        sh """
                            set -x
                            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
                            kubectl get nodes
                        """
                    }
                }
            }
            options {
                timeout(time: 5, unit: 'MINUTES')
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes..."
                    sh """
                        set -x
                        kubectl set image deployment/reactapp-deployment reactapp=${ECR_REPO}:${IMAGE_TAG} || true
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        kubectl apply -f ingress.yaml
                    """
                }
            }
            options {
                timeout(time: 5, unit: 'MINUTES')
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! 🎉"
        }
        failure {
            echo "Pipeline failed. Check the logs above for errors. ❌"
        }
        always {
            echo "Cleaning up workspace..."
            cleanWs()
        }
    }
}
