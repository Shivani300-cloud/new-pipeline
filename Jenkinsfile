pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "shivsoftapp/admin-dashbaord"
        IMAGE_TAG    = "${BUILD_NUMBER}"
        K8S_NS       = "default"
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

        stage('Kubernetes Pre-Check (NO DEPLOY YET)') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KCFG')]) {
                    bat '''
                    set KUBECONFIG=%KCFG%
                    echo ===== K8s CONNECTIVITY CHECK =====
                    kubectl get nodes || exit /b 1
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KCFG')]) {
                    bat '''
                    set KUBECONFIG=%KCFG%

                    kubectl apply -f k8s/deployment.yaml -n %K8S_NS% || exit /b 1
                    kubectl apply -f k8s/service.yaml -n %K8S_NS% || exit /b 1

                    kubectl set image deployment/admin-dashboard admin-dashboard=%DOCKER_IMAGE%:%IMAGE_TAG% -n %K8S_NS% || exit /b 1
                    kubectl rollout status deployment/admin-dashboard -n %K8S_NS% --timeout=180s || exit /b 1
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "🚀 SUCCESS: WINDOWS JENKINS → LOCAL KUBERNETES DEPLOYED"
        }
        failure {
            echo "❌ FAILURE: kubeconfig server IP / network issue (NOT pipeline bug)"
        }
    }
}
