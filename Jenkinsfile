pipeline {
    agent any

    environment {
        IMAGE_NAME = "runtimeterror01/vle7-app"
        IMAGE_TAG  = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/RuntimeTerror001/VLE7.git'
            }
        }

        stage('Build & Test') {
            steps {
                bat 'pip install -r app/requirements.txt'
                bat 'echo No tests yet - VLE7'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-creds') {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    bat 'kubectl apply -f k8s/deployment.yaml'
                    bat 'kubectl apply -f k8s/service.yaml'
                    bat 'kubectl rollout status deployment/vle7-app'
                }
            }
        }
    }

    post {
        success { echo 'VLE7 deployed successfully!' }
        failure { echo 'VLE7 pipeline failed. Check logs.' }
    }
}