pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials-id')
        ROCKY_LINUX_CREDENTIALS = credentials('rocky-linux-ssh-credentials-id')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/boqay/projectA.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("bocoy/project1:${env.BUILD_ID}")
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'DOCKER_CREDENTIALS') {
                        dockerImage.push("${env.BUILD_ID}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Deploy to Rocky Linux') {
            steps {
                script {
                    sshagent(['ROCKY_LINUX_CREDENTIALS']) {
                        sh '''
                            ssh -o StrictHostKeyChecking=no user@rocky-linux-server << EOF
                            docker pull bocoy/project1:${BUILD_ID}
                            docker stop springboot || true
                            docker rm springboot || true
                            docker run -d  --privileged --name springboot -p 9000:8080 bocoy/project1:${BUILD_ID}
                            EOF
                        '''
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}