pipeline {
    agent any

    environment {
        IMAGE_NAME = "shivsoftapp/admin-dashbaord-final"
        IMAGE_TAG  = "035"
    }

    stages {

        stage('Checkout Code from GitLab') {
            steps {
                git branch: 'main',
                url: 'https://gitlab.com/SOFTAPP-TECHNOLOGIES/k8s-jenkins-cicd-pipeline.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Update Kubernetes Image') {
            steps {
                sh '''
                sed -i "s|IMAGE_NAME|$IMAGE_NAME:$IMAGE_TAG|g" k8s/deployment.yaml
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                kubectl get pods
                kubectl get svc
                '''
            }
        }
    }
}
