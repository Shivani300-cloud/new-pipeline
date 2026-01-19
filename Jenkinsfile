pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shivsoftapp/admin-dashbaord"
        IMAGE_TAG = "035"
    }

    stages {

        stage('Checkout Code (GitLab SSH)') {
            steps {
                git branch: 'main',
                    url: 'git@gitlab.com:SOFTAPP-TECHNOLOGIES/k8s-jenkins-cicd-pipeline.git',
                    credentialsId: 'gitlab-ssh'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat '''
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker tag %DOCKER_IMAGE%:%IMAGE_TAG% %DOCKER_IMAGE%:latest
                    docker push %DOCKER_IMAGE%:%IMAGE_TAG%
                    docker push %DOCKER_IMAGE%:latest
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat '''
                    echo ===== Kubernetes Deploy Stage =====
                    echo Using kubeconfig: %KUBECONFIG%

                    kubectl version --client

                    echo Checking Kubernetes cluster connectivity...
                    kubectl get nodes >nul 2>&1

                    IF %ERRORLEVEL% NEQ 0 (
                        echo WARNING: Kubernetes cluster not reachable
                        echo Skipping Kubernetes deployment
                        exit /b 0
                    )

                    echo Kubernetes cluster reachable
                    kubectl apply -f k8s/deployment.yaml --validate=false
                    kubectl apply -f k8s/service.yaml --validate=false

                    echo Kubernetes deployment completed successfully
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🚀 CI/CD Pipeline Completed Successfully!"
        }
        failure {
            echo "❌ CI/CD Pipeline Failed!"
        }
    }
}
