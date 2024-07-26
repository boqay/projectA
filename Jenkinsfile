pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials-id')
        ROCKY_LINUX_CREDENTIALS = credentials('rocky-linux-ssh-credentials-id')
        DOCKER_REGISTRY_URL = 'https://index.docker.io/v1/'
        FIXED_BUILD_ID = "latest"
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
                                sh "./gradlew jib --image=docker.io/${DOCKER_USERNAME}/project1:$FIXED_BUILD_ID"
                                sh "docker pull bocoy/project1:$FIXED_BUILD_ID"
                            }
                        }
                    }
                }
        stage('Deploy to Rocky Linux') {
            steps {
                script {
                    sh "docker stop springboot || true"
                    sh "docker rm springboot || true"
                    sh "docker run -d  --privileged --name springboot -p 8000:8080 bocoy/project1:$FIXED_BUILD_ID"
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