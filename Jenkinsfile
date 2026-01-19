pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shivsoftapp/admin-dashbaord"
        IMAGE_TAG    = "035"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'git@gitlab.com:SOFTAPP-TECHNOLOGIES/k8s-jenkins-cicd-pipeline.git',
                    credentialsId: 'gitlab-ssh'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
            }
        }

        stage('Push Docker Image') {
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

        stage('Deploy to Kubernetes (MAIN STAGE)') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat '''
                    echo ===== KUBERNETES DEPLOYMENT =====
                    set KUBECONFIG=%KUBECONFIG%

                    kubectl version --client || exit /b 1
                    kubectl get nodes || exit /b 1

                    kubectl apply -f k8s/deployment.yaml || exit /b 1
                    kubectl apply -f k8s/service.yaml || exit /b 1

                    kubectl rollout status deployment/admin-dashboard --timeout=180s || exit /b 1

                    kubectl get pods
                    kubectl get svc
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🚀 CI/CD SUCCESS – DOCKER + KUBERNETES DEPLOYED"
        }
        failure {
            echo "❌ PIPELINE FAILED – CHECK KUBECONFIG / CLUSTER"
        }
    }
}
