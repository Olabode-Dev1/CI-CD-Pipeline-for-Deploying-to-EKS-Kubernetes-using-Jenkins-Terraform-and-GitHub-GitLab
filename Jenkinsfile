pipeline {
    agent any
    environment {
        AWS_REGION = 'us-west-2'  // EDIT: Your region
        EKS_CLUSTER = 'your-eks-cluster'  // EDIT: Your cluster name
        APP_NAME = 'nodejs-app'  // ← Matches deployment.yaml
        ECR_REGISTRY = '123456789012.dkr.ecr.us-west-2.amazonaws.com'  // EDIT: Your ECR repo
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/eks-demo.git'  // EDIT
            }
        }
        stage('Build & Push') {
            steps {
                script {
                    // Login to ECR
                    sh "aws ecr get-login-password | docker login --username AWS --password-stdin ${ECR_REGISTRY}"
                    
                    // Build and push
                    docker.build("${ECR_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}")
                    docker.withRegistry("https://${ECR_REGISTRY}") {
                        docker.image("${ECR_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                script {
                    // Update kubectl config
                    sh "aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER}"
                    
                    // Deploy with build number
                    sh """
                    sed -i 's#YOUR_ECR_REGISTRY#${ECR_REGISTRY}#g' k8s/deployment.yaml
                    sed -i 's#BUILD_NUMBER#${BUILD_NUMBER}#g' k8s/deployment.yaml
                    kubectl apply -f k8s/
                    kubectl rollout status deployment/${APP_NAME}
                    """
                }
            }
        }
    }
}
