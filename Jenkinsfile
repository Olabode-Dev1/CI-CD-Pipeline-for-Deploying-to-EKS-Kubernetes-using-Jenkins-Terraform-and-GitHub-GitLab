// Jenkinsfile

pipeline {
    agent any
    
    environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
        AWS_REGION            = 'us-west-2'
        CLUSTER_NAME          = 'my-eks-cluster'
        DOCKER_REGISTRY       = 'your-docker-registry'
        APP_NAME              = 'my-app'
        TERRAFORM_DIR         = 'infrastructure'
        KUBE_CONFIG           = credentials('kube-config')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            when {
                branch 'main'
            }
            steps {
                dir(env.TERRAFORM_DIR) {
                    sh 'terraform init'
                }
            }
        }
        
        stage('Terraform Plan') {
            when {
                branch 'main'
            }
            steps {
                dir(env.TERRAFORM_DIR) {
                    sh 'terraform plan -out=tfplan'
                }
            }
        }
        
        stage('Terraform Apply') {
            when {
                branch 'main'
            }
            steps {
                dir(env.TERRAFORM_DIR) {
                    sh 'terraform apply -auto-approve tfplan'
                }
            }
        }
        
        stage('Configure kubectl') {
            steps {
                script {
                    // Update kubeconfig with the newly created cluster
                    sh """
                    aws eks --region ${AWS_REGION} update-kubeconfig --name ${CLUSTER_NAME}
                    kubectl config use-context arn:aws:eks:${AWS_REGION}:${AWS_ACCOUNT_ID}:cluster/${CLUSTER_NAME}
                    """
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://${DOCKER_REGISTRY}', 'docker-registry-cred') {
                        docker.image("${DOCKER_REGISTRY}/${APP_NAME}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
        stage('Deploy to EKS') {
            steps {
                script {
                    // Apply Kubernetes manifests
                    sh "kubectl apply -f kubernetes/deployment.yaml"
                    sh "kubectl apply -f kubernetes/service.yaml"
                    
                    // Check rollout status
                    sh "kubectl rollout status deployment/${APP_NAME}"
                }
            }
        }
        
        stage('Smoke Test') {
            steps {
                script {
                    // Simple smoke test to verify deployment
                    def serviceUrl = sh(script: "kubectl get service ${APP_NAME} -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'", returnStdout: true).trim()
                    sh "curl -sSf http://${serviceUrl}/health"
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            slackSend channel: '#ci-cd', message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} failed"
        }
        success {
            slackSend channel: '#ci-cd', message: "Build ${env.JOB_NAME} #${env.BUILD_NUMBER} succeeded"
        }
    }
}