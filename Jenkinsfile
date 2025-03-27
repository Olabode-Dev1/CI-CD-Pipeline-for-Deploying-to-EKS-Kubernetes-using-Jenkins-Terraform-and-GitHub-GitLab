pipeline {
    agent any
    
    // --- ENVIRONMENT VARIABLES (EDIT THESE) ---
    environment {
        // AWS Region where EKS lives
        AWS_REGION = 'us-west-2'  // EDIT: Your EKS region (e.g., 'eu-central-1')
        
        // Name of your EKS cluster
        EKS_CLUSTER = 'my-eks-cluster'  // EDIT: Get this from `aws eks list-clusters`
        
        // Docker registry path (ECR or Docker Hub)
        DOCKER_REGISTRY = '123456789012.dkr.ecr.us-west-2.amazonaws.com'  // EDIT: Your ECR repo URL
        APP_NAME = 'my-app'  // EDIT: Your application name
    }
    
    stages {
        // --- STAGE 1: Checkout Code ---
        stage('Checkout') {
            steps {
                git branch: 'main',  // EDIT: Your Git branch (e.g., 'dev', 'production')
                      url: 'https://github.com/your-username/your-repo.git'  // EDIT: Your repo URL
            }
        }
        
        // --- STAGE 2: Build Docker Image ---
        stage('Build') {
            steps {
                script {
                    // Builds image with tag like: 123456789012.dkr.ecr.us-west-2.amazonaws.com/my-app:42
                    docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}")
                }
            }
        }
        
        // --- STAGE 3: Push to Container Registry ---
        stage('Push to ECR') {
            steps {
                script {
                    // Uses Jenkins credentials ID 'ecr-credentials' (set this up in Jenkins)
                    docker.withRegistry("https://${DOCKER_REGISTRY}", 'ecr-credentials') {
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        // --- STAGE 4: Deploy to EKS ---
        stage('Deploy to EKS') {
            steps {
                script {
                    // Authenticates kubectl with EKS
                    sh """
                    aws eks --region ${AWS_REGION} update-kubeconfig --name ${EKS_CLUSTER}
                    kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_REGISTRY}/${APP_NAME}:${BUILD_NUMBER} -n default
                    kubectl rollout status deployment/${APP_NAME} -n default
                    """
                }
            }
        }
    }
    
    // --- POST-BUILD ACTIONS ---
    post {
        failure {
            slackSend channel: '#ci-alerts',  // EDIT: Your Slack channel
                     message: "FAILED: Job '${env.JOB_NAME}' (Build ${env.BUILD_NUMBER})"
        }
        success {
            slackSend channel: '#ci-alerts',
                     message: "SUCCESS: Job '${env.JOB_NAME}' (Build ${env.BUILD_NUMBER})"
        }
    }
}
