pipeline {
    agent any

    environment {
        IMAGE_NAME = "shivsoftapp/admin-dashbaord"
        IMAGE_TAG  = "039"
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
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Kubernetes Image in YAML') {
            steps {
                sh """
                IMAGE_FULL="${IMAGE_NAME}:${IMAGE_TAG}"
                # Replace placeholder IMAGE_NAME in deployment.yaml with the new image
                sed -i'' -e "s|IMAGE_NAME|$IMAGE_FULL|g" k8s/deployment.yaml
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh """
                    export KUBECONFIG=\$KUBECONFIG
                    kubectl apply -f k8s/deployment.yaml
                    """
                }
            }
        }

    } // end of stages

    post {
        success {
            echo "Deployment successful!"
        }
        failure {
            echo "Deployment failed!"
        }
    }
}
