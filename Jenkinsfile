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
                sh """
                echo "Docker Image : ${IMAGE_NAME}"
                echo "Build Number : ${BUILD_NUMBER}"
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('app') {
                    withSonarQubeEnv('SonarQube') {
                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=springboot-app \
                        -Dsonar.projectName=springboot-app
                        '''
                    }
                }
            }
        }
                   
        stage('Docker Build') {
            steps {
                dir('app') {
                    sh """
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} ${IMAGE_NAME}:latest
                    """
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
                    sh """
                    echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin

                    docker push ${IMAGE_NAME}:${BUILD_NUMBER}
                    docker push ${IMAGE_NAME}:latest

                    docker logout
                    """
                }
            }
        }

        stage('Debug Kubernetes') {
            environment {
                KUBECONFIG = "/var/lib/jenkins/.kube/config"
            }
            steps {
                sh '''
                echo "======================================="
                echo "Current User:"
                whoami

                echo ""
                echo "HOME:"
                echo $HOME

                echo ""
                echo "KUBECONFIG:"
                echo $KUBECONFIG

                echo ""
                echo "Kubectl Path:"
                which kubectl

                echo ""
                echo "Kubectl Client Version:"
                kubectl version --client

                echo ""
                echo "Current Context:"
                kubectl config current-context

                echo ""
                echo "Cluster Info:"
                kubectl cluster-info

                echo ""
                echo "Nodes:"
                kubectl get nodes

                echo ""
                echo "Kube Directory:"
                ls -la /var/lib/jenkins/.kube

                echo ""
                echo "======================================="
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            environment {
                KUBECONFIG = "/var/lib/jenkins/.kube/config"
            }
            steps {
                sh '''
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml

                kubectl set image deployment/springboot-app \
                springboot-app=${IMAGE_NAME}:${BUILD_NUMBER}

                kubectl rollout status deployment/springboot-app
                '''
            }
        }

        stage('Verify Deployment') {
            environment {
                KUBECONFIG = "/var/lib/jenkins/.kube/config"
            }
            steps {
                sh '''
                kubectl get deployments
                kubectl get pods -o wide
                kubectl get svc
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker image prune -f'
            }
        }
    }

    post {
        success {
            echo "===================================="
            echo "CI Pipeline Completed Successfully"
            echo "Docker Image: ${IMAGE_NAME}:${BUILD_NUMBER}"
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