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
                withCredentials([string(credentialsId: 'kubeconfig', variable: 'KUBECONFIG_CONTENT')]) {
                    bat '''
                        powershell -Command "$env:KUBECONFIG_CONTENT | Out-File -FilePath kubeconfig-temp.yaml -Encoding utf8"
                    '''
                    bat 'kubectl --kubeconfig=kubeconfig-temp.yaml apply -f k8s/deployment.yaml'
                    bat 'kubectl --kubeconfig=kubeconfig-temp.yaml apply -f k8s/service.yaml'
                    bat 'kubectl --kubeconfig=kubeconfig-temp.yaml rollout status deployment/vle7-app'
                    bat 'del kubeconfig-temp.yaml'
                }
            }
        }
    }

    post {
        success { echo 'VLE7 deployed successfully!' }
        failure { echo 'VLE7 pipeline failed. Check logs.' }
    }
}