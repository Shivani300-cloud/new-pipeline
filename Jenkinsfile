pipeline {
    agent any

    environment {
        IMAGE_NAME = "shivanitusharsharma/admin-dashbaord"
        IMAGE_TAG  = "039"
        IMAGE_FULL = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Code from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Shivani300-cloud/new-pipeline.git'
            }
        }

        stage('Docker Build') {
            steps {
                sh """
                docker build -t ${IMAGE_FULL} .
                """
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds-new',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                    docker push ${IMAGE_FULL}
                    """
                }
            }
        }

        stage('Update Kubernetes Image in YAML') {
            steps {
                sh """
                sed -i'' -e "s|IMAGE_NAME|${IMAGE_FULL}|g" k8s/deployment.yaml
                """
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                    export KUBECONFIG=\$KUBECONFIG_FILE
                    kubectl apply -f k8s/deployment.yaml
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed!"
        }
    }
}
