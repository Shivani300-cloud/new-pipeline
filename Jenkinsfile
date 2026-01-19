pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shivsoftapp/admin-dashbaord"
        IMAGE_TAG = "035"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'git@gitlab.com:SOFTAPP-TECHNOLOGIES/k8s-jenkins-cicd-pipeline.git',
                    credentialsId: 'gitlab-ssh'
            }
        }

        stage('Build Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat '''
                    set KUBECONFIG=%KUBECONFIG%

                    kubectl get nodes || exit /b 1

                    kubectl apply -f k8s/deployment.yaml || exit /b 1
                    kubectl apply -f k8s/service.yaml || exit /b 1

                    kubectl rollout status deployment/admin-dashboard-deployment --timeout=120s || exit /b 1
                    '''
                }
            }
        }
    }
}
