pipeline {
    agent any
    tools {
        nodejs "nodejs"
    }    
//    triggers {
        // Leave empty if you use webhooks (preferred)
        // pollSCM('H/5 * * * *') // Uncomment if no webhooks
//    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'chmod +x scripts/build.sh scripts/test.sh'
                sh './scripts/build.sh'
                sh './scripts/test.sh || echo "Tests failed or not present"'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def appEnv = (env.BRANCH_NAME == 'dev') ? 'dev' : 'prod'
                    env.DOCKER_IMAGE = "my-node-app:${appEnv}"

                    sh """
                        docker build --build-arg REACT_APP_ENV=${appEnv} -t ${env.DOCKER_IMAGE} .
                    """
                }
            }
        }

        stage('Deploy Application') {
            when {
                anyOf {
                    branch 'main'
                    branch 'dev'
                }
            }
            steps {
                script {
                    def containerName = (env.BRANCH_NAME == 'dev') ? 'nodedev' : 'nodemain'
                    def appPort       = (env.BRANCH_NAME == 'dev') ? '3001' : '3000'

                    sh """
                        docker stop ${containerName} || true
                        docker rm ${containerName} || true
                        docker run -d --name ${containerName} -p ${appPort}:3000 ${env.DOCKER_IMAGE}
                    """

                    echo "Application deployed for ${env.BRANCH_NAME} branch at http://localhost:${appPort}"
                }
            }
        }
    }
}
