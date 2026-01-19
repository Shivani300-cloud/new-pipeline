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
                bat 'docker build -t %DOCKER_IMAGE%:%IMAGE_TAG% .'
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
                    echo ================================
                    echo  KUBERNETES DEPLOYMENT (FINAL)
                    echo ================================

                    set KUBECONFIG=%KUBECONFIG%

                    kubectl version --client || exit /b 1

                    echo Checking cluster connectivity...
                    kubectl get nodes || exit /b 1

                    echo Applying Deployment...
                    kubectl apply -f k8s/deployment.yaml || exit /b 1

                    echo Applying Service...
                    kubectl apply -f k8s/service.yaml || exit /b 1

                    echo Waiting for rollout...
                    kubectl rollout status deployment/admin-dashboard-deployment --timeout=120s || exit /b 1

                    echo ✅ DEPLOYMENT & SERVICE SUCCESSFUL
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🚀 CI/CD PIPELINE SUCCESSFULLY COMPLETED"
        }
        failure {
            echo "❌ PIPELINE FAILED - CHECK KUBERNETES CONNECTIVITY"
        }
    }
}
