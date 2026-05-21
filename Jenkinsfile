pipeline {
    agent any

    environment {
        ECR_REGISTRY = "051602877417.dkr.ecr.us-east-1.amazonaws.com"
        AWS_REGION   = "us-east-1"
        CLUSTER_NAME = "three-tier-cluster"
    }

    stages {

        stage('Checkout') {
            steps {
                git url: "https://github.com/AnushkaSri1/TWSThreeTierAppChallenge.git",
                    branch: "main"
            }
        }

        stage('ECR Login') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials'
                ]]) {
                    sh '''
                        aws ecr get-login-password --region $AWS_REGION \
                        | docker login --username AWS --password-stdin $ECR_REGISTRY
                    '''
                }
            }
        }

        stage('Build Frontend') {
            steps {
                sh '''
                    docker build -t three-tier-frontend Application-Code/frontend
                    docker tag three-tier-frontend:latest $ECR_REGISTRY/three-tier-frontend:latest
                    docker push $ECR_REGISTRY/three-tier-frontend:latest
                '''
            }
        }

        stage('Build Backend') {
            steps {
                sh '''
                    docker build -t three-tier-backend Application-Code/backend
                    docker tag three-tier-backend:latest $ECR_REGISTRY/three-tier-backend:latest
                    docker push $ECR_REGISTRY/three-tier-backend:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    aws eks update-kubeconfig --name $CLUSTER_NAME --region $AWS_REGION
                    kubectl apply -f Kubernetes-Manifests-file/Database/secrets.yaml
                    kubectl apply -f Kubernetes-Manifests-file/Database/pvc.yaml
                    kubectl apply -f Kubernetes-Manifests-file/Database/service.yaml
                    kubectl apply -f Kubernetes-Manifests-file/Database/deployment.yaml
                    kubectl apply -f Kubernetes-Manifests-file/Backend/service.yaml
                    kubectl apply -f Kubernetes-Manifests-file/Backend/deployment.yaml
                    kubectl apply -f Kubernetes-Manifests-file/Frontend/service.yaml
                    kubectl apply -f Kubernetes-Manifests-file/Frontend/deployment.yaml
                    kubectl apply -f Kubernetes-Manifests-file/ingress.yaml
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs above.'
        }
    }
}
