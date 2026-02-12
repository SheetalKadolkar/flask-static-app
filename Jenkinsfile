pipeline {
    agent any

    environment {
        IMAGE = "sheetalkadolkar/flask-app"
        TAG = "${BUILD_NUMBER}"
        CLUSTER = "flask-cluster"
        REGION = "us-east-1"
    }

    stages {

        stage('Checkout') {
            steps {
                git 'https://github.com/SheetalKadolkar/flask-static-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE:$TAG .'
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                }
            }
        }

        stage('Push Image') {
            steps {
                sh 'docker push $IMAGE:$TAG'
            }
        }

        stage('Update Kubeconfig') {
            steps {
                sh 'aws eks update-kubeconfig --region $REGION --name $CLUSTER'
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                sed -i "s|IMAGE_PLACEHOLDER|$IMAGE:$TAG|g" k8s/deployment.yaml
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
