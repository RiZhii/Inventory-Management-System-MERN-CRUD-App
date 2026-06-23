pipeline {
agent any

```
environment {
    AWS_REGION  = 'us-east-1'
    AWS_ACCOUNT = '899867382718'

    BACKEND_REPO  = "${AWS_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com/inventory-backend"
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
            docker build \
            -t inventory-backend:${BUILD_NUMBER} \
            -f Backend/dockerfile \
            Backend
            '''
        }
    }

    stage('Build Frontend Image') {
        steps {
            sh '''
            docker build \
            -t inventory-frontend:${BUILD_NUMBER} \
            -f Frontend/inventory_management_system/dockerfile \
            Frontend/inventory_management_system
            '''
        }
    }

    stage('Login To ECR') {
        steps {
            sh '''
            aws ecr get-login-password --region us-east-1 | \
            docker login --username AWS \
            --password-stdin 899867382718.dkr.ecr.us-east-1.amazonaws.com
            '''
        }
    }

    stage('Push Backend Image') {
        steps {
            sh '''
            docker tag inventory-backend:${BUILD_NUMBER} $BACKEND_REPO:latest
            docker push $BACKEND_REPO:latest
            '''
        }
    }

    stage('Push Frontend Image') {
        steps {
            sh '''
            docker tag inventory-frontend:${BUILD_NUMBER} $FRONTEND_REPO:latest
            docker push $FRONTEND_REPO:latest
            '''
        }
    }

    stage('Deploy To DEV') {
        when {
            branch 'develop'
        }

        steps {
            sshagent(credentials: ['dev-server-key']) {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@10.0.3.243 "

                aws ecr get-login-password --region us-east-1 | \
                docker login --username AWS \
                --password-stdin 899867382718.dkr.ecr.us-east-1.amazonaws.com

                cd /opt/inventory-app

                docker compose pull

                docker compose down

                docker compose up -d
                "
                '''
            }
        }
    }

    stage('Deploy To STAGING') {
        when {
            branch 'staging'
        }

        steps {
            sshagent(credentials: ['staging-server-key']) {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@10.0.4.166 "

                aws ecr get-login-password --region us-east-1 | \
                docker login --username AWS \
                --password-stdin 899867382718.dkr.ecr.us-east-1.amazonaws.com

                cd /opt/inventory-app

                docker compose pull

                docker compose down

                docker compose up -d
                "
                '''
            }
        }
    }

    stage('Deploy To PROD') {
        when {
            branch 'main'
        }

        steps {
            sshagent(credentials: ['prod-server-key']) {
                sh '''
                ssh -o StrictHostKeyChecking=no ubuntu@10.0.5.238 "

                aws ecr get-login-password --region us-east-1 | \
                docker login --username AWS \
                --password-stdin 899867382718.dkr.ecr.us-east-1.amazonaws.com

                cd /opt/inventory-app

                docker compose pull

                docker compose down

                docker compose up -d
                "
                '''
            }
        }
    }
}

post {
    always {
        cleanWs()
    }

    success {
        echo 'Pipeline Completed Successfully'
    }

    failure {
        echo 'Pipeline Failed'
    }
}
```

}
