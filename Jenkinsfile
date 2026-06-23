pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_ACCOUNT = '899867382718'

        BACKEND_REPO = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/inventory-backend"
        FRONTEND_REPO = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/inventory-frontend"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend Image') {
            steps {
                sh '''
                docker build -t inventory-backend:${BUILD_NUMBER} \
                -f Backend/dockerfile Backend
                '''
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh '''
                docker build -t inventory-frontend:${BUILD_NUMBER} \
                -f Frontend/inventory_management_system/dockerfile \
                Frontend/inventory_management_system
                '''
            }
        }

        stage('Login To ECR') {
            steps {
                withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds']
                ]) {
                    sh '''
                    aws sts get-caller-identity

                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS \
                    --password-stdin ${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com
                    '''
                }
            }
        }

        stage('Push Backend Image') {
            steps {
                sh '''
                docker tag inventory-backend:${BUILD_NUMBER} ${BACKEND_REPO}:latest
                docker push ${BACKEND_REPO}:latest
                '''
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh '''
                docker tag inventory-frontend:${BUILD_NUMBER} ${FRONTEND_REPO}:latest
                docker push ${FRONTEND_REPO}:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {

                    def SERVER_IP = ""
                    def SSH_CRED = ""

                    if (env.BRANCH_NAME == "develop") {
                        SERVER_IP = "10.0.3.243"
                        SSH_CRED = "dev-server-key"
                    }
                    else if (env.BRANCH_NAME == "staging") {
                        SERVER_IP = "10.0.4.166"
                        SSH_CRED = "staging-server-key"
                    }
                    else if (env.BRANCH_NAME == "prod") {
                        SERVER_IP = "10.0.5.238"
                        SSH_CRED = "prod-server-key"
                    }

                    sshagent([SSH_CRED]) {

                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} '

                        aws ecr get-login-password --region us-east-1 | \
                        docker login --username AWS \
                        --password-stdin ${AWS_ACCOUNT}.dkr.ecr.us-east-1.amazonaws.com

                        docker pull ${BACKEND_REPO}:latest
                        docker pull ${FRONTEND_REPO}:latest

                        docker stop inventory-backend || true
                        docker rm inventory-backend || true

                        docker stop inventory-frontend || true
                        docker rm inventory-frontend || true

                        docker run -d \
                        --name inventory-backend \
                        -p 3001:3001 \
                        ${BACKEND_REPO}:latest

                        docker run -d \
                        --name inventory-frontend \
                        -p 80:80 \
                        ${FRONTEND_REPO}:latest

                        docker ps
                        '
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}