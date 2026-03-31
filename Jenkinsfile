pipeline {
    agent any

    environment {
        AWS_REGION = "us-west-2"
        AWS_ACCOUNT_ID = "533267244722"
        ECR_REPO = "workshop"
    }

    stages {

        stage('Build Docker Images') {
            steps {
                sh 'docker build -t frontend ./Application-Code/frontend'
                sh 'docker build -t backend ./Application-Code/backend'
            }
        }

        stage('Login to AWS ECR') {
            steps {
                sh '''
                aws ecr get-login-password --region $AWS_REGION \
                | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com
                '''
            }
        }

        stage('Tag and Push Images') {
            steps {
                sh '''
                docker tag frontend:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:frontend
                docker tag backend:latest $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:backend

                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:frontend
                docker push $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$ECR_REPO:backend
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f Kubernetes-Manifest-file/'
            }
        }
    }
}
