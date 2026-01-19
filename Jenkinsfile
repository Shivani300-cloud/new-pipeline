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
                bat '''
                docker build -t %IMAGE_NAME%:%IMAGE_TAG% .
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
                    bat '''
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker push %IMAGE_NAME%:%IMAGE_TAG%
                    '''
                }
            }
        }

        stage('Update Kubernetes Image in YAML') {
            steps {
                powershell '''
                $image = "$env:IMAGE_NAME`:$env:IMAGE_TAG"
                (Get-Content k8s\\deployment.yaml) `
                  -replace "IMAGE_NAME", $image |
                Set-Content k8s\\deployment.yaml
                '''
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
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '''
                kubectl get pods
                kubectl get svc
                '''
            }
        }
    }
}
