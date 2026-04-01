pipeline {
    agent {
        label '3-tier'
    }

    environment {
        ECR_REGISTRY = "533267244722.dkr.ecr.us-west-2.amazonaws.com"
        AWS_REGION   = "us-west-2"
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

        stage('Build Frontend Docker Image') {
            steps {
                sh '''
                    docker build -t workshop-frontend Application-Code/frontend
                    docker tag workshop-frontend:latest $ECR_REGISTRY/workshop:frontend
                    docker push $ECR_REGISTRY/workshop:frontend
                '''
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                sh '''
                    docker build -t workshop-backend Application-Code/backend
                    docker tag workshop-backend:latest $ECR_REGISTRY/workshop:backend
                    docker push $ECR_REGISTRY/workshop:backend
                '''
            }
        }

        stage('Deploy Database') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {
                    sh '''
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/database/secrets.yaml
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/database/pv.yaml
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/database/pvc.yaml
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/database/deployment.yaml
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/database/service.yaml
                    '''
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {
                    sh '''
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/backend/deployment.yaml
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/backend/service.yaml
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {
                    sh '''
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/frontend/deployment.yaml
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/frontend/service.yaml
                    '''
                }
            }
        }

        stage('Apply Ingress') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {
                    sh '''
                        kubectl --kubeconfig=$KUBE_CONFIG apply -f Kubernetes-Manifest-file/ingress.yaml
                    '''
                }
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
