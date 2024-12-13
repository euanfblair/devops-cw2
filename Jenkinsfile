pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'euanfblair/cw2-server'
        VERSION = "${env.BUILD_NUMBER}"
        DOCKER_CREDENTIALS = 'dockerhub-credentials' 
        PRODUCTION_SERVER = 'ubuntu@<Production_Server_IP>'
        SSH_CREDENTIALS = 'production-server-ssh'    
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Checking out code...'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE}:${VERSION} ."
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                echo 'Pushing Docker image to DockerHub...'
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                    sh "docker push ${DOCKER_IMAGE}:${VERSION}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                sshagent(credentials: ["${SSH_CREDENTIALS}"]) {
                    sh """
                    ssh ${PRODUCTION_SERVER} \\
                    'kubectl set image deployment/cw2-server cw2-server=${DOCKER_IMAGE}:${VERSION}'
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}
