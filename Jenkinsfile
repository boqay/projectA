pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials-id')
        ROCKY_LINUX_CREDENTIALS = credentials('rocky-linux-ssh-credentials-id')
        DOCKER_REGISTRY_URL = 'https://index.docker.io/v1/'
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
                            withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                print 'id :' + DOCKER_USERNAME
                                print 'pw :' + DOCKER_PASSWORD
                                sh '''
                                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin $DOCKER_REGISTRY_URL
                                '''
                                sh "chmod +x ./gradlew"
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
                            docker pull bocoy/project1:${BUILD_ID}
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