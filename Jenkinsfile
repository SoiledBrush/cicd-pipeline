pipeline {
    agent any
    tools {
        nodejs "nodejs"
    }

    environment {
        IMAGE_NAME = "my-node-app"
        IMAGE_TAG = "${env.BRANCH_NAME}"
        PORT = "${env.BRANCH_NAME == 'main' ? '3000' : '3001'}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}", ".")
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                docker rm -f ${IMAGE_NAME}-${BRANCH_NAME} || true
                docker run -d --name ${IMAGE_NAME}-${BRANCH_NAME} -p ${PORT}:3000 ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
    }
}
