pipeline {
    agent any

    environment {
        DOCKER_HUB_REPO = "abdallah1312/ivolve"
        KUBERNETES_DEPLOYMENT_FILE = "deployment.yaml"
        DOCKER_CREDENTIALS = "Docker-cred"
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Update Deployment YAML') {
            steps {
                script {
                    sh 'sed -i \'s|image: .*|image: ${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}|\' ${KUBERNETES_DEPLOYMENT_FILE}'
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh "kubectl apply -f ${KUBERNETES_DEPLOYMENT_FILE}"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check logs for details.'
        }
        always {
            echo "Cleaning up resources..."
            sh "docker rmi ${DOCKER_HUB_REPO}:${env.BUILD_NUMBER} || true"
        }
    }
}
