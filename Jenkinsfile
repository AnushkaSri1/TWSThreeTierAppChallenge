pipeline {
    agent {
	label '3-tier'
}

   
    stages {

        stage('Checkout') {
            steps {
                script {
                    git url: "https://github.com/AnushkaSri1/TWSThreeTierAppChallenge.git", branch: "main"
                }
            }
        }

       stage('Build Frontend Docker Image') {
 	   steps {
        	script {
            	     withCredentials([[
               		 $class: 'AmazonWebServicesCredentialsBinding',
                	credentialsId: 'aws-credentials'
            ]]) {

                sh '''
                aws configure set region us-west-2
                aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 533267244722.dkr.ecr.us-west-2.amazonaws.com

                docker build -t workshop-frontend Application-Code/frontend
                docker tag workshop-frontend:latest 533267244722.dkr.ecr.us-west-2.amazonaws.com/workshop:frontend
                docker push 533267244722.dkr.ecr.us-west-2.amazonaws.com/workshop:frontend
                '''
            }
        }
    }
}
        stage('Build Backend Docker Image') {
            steps {
                script {
                    sh "docker build -t workshop-backend Application-Code/backend"
                    sh "docker tag workshop-backend:latest 533267244722.dkr.ecr.us-west-2.amazonaws.com/workshop:backend"
                    sh "docker push 533267244722.dkr.ecr.us-west-2.amazonaws.com/workshop:backend"
                }
            }
        }

        stage('Deploy Database') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {

                        sh """
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/database/secrets.yaml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/database/pv.yaml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/database/pvc.yaml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/database/deployment.yaml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/database/service.yaml
                        """
                    }
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {

                        sh """
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/backend/deployment.yaml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/backend/service.yaml
                        """
                    }
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {

                        sh """
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/frontend/deployment.yaml
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/frontend/service.yaml
                        """
                    }
                }
            }
        }

        stage('Apply Ingress') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG')]) {

                        sh """
                        kubectl --kubeconfig=${KUBE_CONFIG} apply -f Kubernetes-Manifest-file/ingress.yaml
                        """
                    }
                }
            }
        }

    }
}
