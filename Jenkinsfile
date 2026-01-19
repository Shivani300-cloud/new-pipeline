pipeline {
    agent any

    environment {
        IMAGE_NAME = "shivsoftapp/admin-dashboard"
        IMAGE_TAG  = "${BUILD_NUMBER}"
        KUBECONFIG = "C:\\ProgramData\\Jenkins\\.kube\\config"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat """
                docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %IMAGE_NAME%:%IMAGE_TAG%
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat """
                kubectl get nodes || exit /b 1

                kubectl set image deployment/admin-dashboard-deployment ^
                admin-dashboard=%IMAGE_NAME%:%IMAGE_TAG% || exit /b 1

                kubectl rollout status deployment/admin-dashboard-deployment --timeout=120s || exit /b 1
                """
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD Pipeline SUCCESSFULLY COMPLETED"
        }
        failure {
            echo "❌ CI/CD Pipeline FAILED"
        }
    }
}
