pipeline {
    agent any

    environment {
        IMAGE_NAME = "dilwarahmed/springboot-cicd"
        DOCKER_CREDENTIALS = "dockerhub-creds"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Dilwarahmed2425/aws-devops-cicd-pipeline.git'
            }
        }

        stage('Build') {
            steps {
                dir('app') {
                    sh 'mvn clean package'
                }
            }
        }

        stage('Test') {
            steps {
                dir('app') {
                    sh 'mvn test'
                }
            }
        }

        stage('Print Variables') {
            steps {
                sh '''
                echo "Docker Image : ${IMAGE_NAME}"
                echo "Build Number : ${BUILD_NUMBER}"
                '''
            }
        }

        stage('Docker Build') {
            steps {
                dir('app') {
                    sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                    '''
                }
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest

                    docker logout
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                sh '''
                docker image prune -f
                '''
            }
        }
    }

    post {

        success {
            echo "===================================="
            echo "CI Pipeline Completed Successfully"
            echo "Image: ${IMAGE_NAME}:${BUILD_NUMBER}"
            echo "===================================="
        }

        failure {
            echo "===================================="
            echo "CI Pipeline Failed"
            echo "Check Jenkins Console Output"
            echo "===================================="
        }

        always {
            cleanWs()
        }
    }
}