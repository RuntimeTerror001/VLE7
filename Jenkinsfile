pipeline {
    agent any

    environment {
        IMAGE_NAME = "runtimeterror01/vle7-app"
        IMAGE_TAG  = "v${BUILD_NUMBER}"
        KUBECONFIG = "C:\\Users\\${env.USERNAME}\\.kube\\config"
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
                bat 'kubectl --kubeconfig="%USERPROFILE%\\.kube\\config" config use-context docker-desktop'
                bat 'kubectl --kubeconfig="%USERPROFILE%\\.kube\\config" apply -f k8s/deployment.yaml --validate=false'
                bat 'kubectl --kubeconfig="%USERPROFILE%\\.kube\\config" apply -f k8s/service.yaml --validate=false'
                bat 'kubectl --kubeconfig="%USERPROFILE%\\.kube\\config" rollout status deployment/vle7-app'
            }
        }
    }

    post {
        success { echo 'VLE7 deployed successfully!' }
        failure { echo 'VLE7 pipeline failed. Check logs.' }
    }
}