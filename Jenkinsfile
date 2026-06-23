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
    }

    post {
        always {
            cleanWs()
        }
    }
}