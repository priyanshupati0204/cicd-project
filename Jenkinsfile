pipeline {
    agent any

    environment {
        IMAGE = 'priyanshu0204/cicd-project'
        TAG   = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Test') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --default-timeout=100 -r requirements.txt
                    pytest
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE:$TAG .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                  usernameVariable: 'DH_USER',
                                                  passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker push $IMAGE:$TAG
                        docker logout
                    '''
                }
            }
        }

        stage('Minikube Deployment') {
            steps {
                sh '''
                    sed "s|IMAGE_TAG|$TAG|" k8s/deployment.yaml | kubectl apply -f -
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/cicd-app --timeout=180s
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get deployment
                    kubectl get svc
                '''
            }
        }
    }
}