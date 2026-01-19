pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shivsoftapp/admin-dashbaord"
        IMAGE_TAG    = "035"
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

        /* ===============================
           KUBERNETES DEPLOYMENT STAGE
           =============================== */
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    bat '''
                    echo ===============================
                    echo   Kubernetes Deployment Stage
                    echo ===============================

                    set KUBECONFIG=%KUBECONFIG%

                    kubectl version --client

                    echo Checking cluster connectivity...
                    kubectl get nodes

                    IF %ERRORLEVEL% NEQ 0 (
                        echo ❌ Kubernetes cluster not reachable
                        exit /b 1
                    )

                    echo ✅ Cluster reachable

                    echo Applying Deployment...
                    kubectl apply -f k8s/deployment.yaml --validate=false

                    echo Applying Service...
                    kubectl apply -f k8s/service.yaml --validate=false

                    echo Waiting for rollout...
                    kubectl rollout status deployment/admin-dashboard-deployment

                    echo ✅ Kubernetes deployment successful
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
