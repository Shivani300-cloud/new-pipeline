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
                # For Linux compatibility
                sed -i -e "s|IMAGE_NAME|${IMAGE_FULL}|g" k8s/deployment.yaml
                """
            }
        }

        stage('Verify Kubeconfig & Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_FILE')]) {
                    sh """
                    export KUBECONFIG=\$KUBECONFIG_FILE

                    echo "🔹 Testing Kubernetes cluster connectivity..."
                    if ! kubectl cluster-info; then
                        echo "❌ ERROR: Cannot reach Kubernetes cluster. Check kubeconfig!"
                        exit 1
                    fi

                    echo "🔹 Deploying application..."
                    kubectl apply --validate=false -f k8s/deployment.yaml

                    # Replace <your-deployment-name> with actual deployment name in YAML
                    echo "🔹 Waiting for deployment rollout..."
                    kubectl rollout status deployment/<your-deployment-name> || {
                        echo "❌ Deployment rollout failed"
                        exit 1
                    }

                    echo "✅ Deployment completed successfully!"
                    """
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline finished successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}
