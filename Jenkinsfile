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
        stage('Build with Gradle and Jib') {
                    steps {
                        script {
                            // Jib을 사용하여 Docker 이미지 빌드 및 푸시
                            withCredentials([usernamePassword(credentialsId: 'DOCKER_CREDENTIALS', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                sh "./gradlew jib --image=docker.io/${DOCKER_USERNAME}/project1:${env.BUILD_ID}"
                            }
                        }
                    }
                }
        stage('Deploy to Rocky Linux') {
            steps {
                script {
                    sshagent(['ROCKY_LINUX_CREDENTIALS']) {
                        sh '''
                            ssh -p 1818 -o StrictHostKeyChecking=no root@127.0.0.1 << EOF
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