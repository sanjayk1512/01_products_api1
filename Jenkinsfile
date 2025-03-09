pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
        IMAGE_NAME = 'darin04/products-api'
        K8S_NAMESPACE = 'default'
    }
    stages {
        stage('Clone Repository') {
            steps {
                git 'https://github.com/Darin40/01_products_api.git'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def appVersion = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
                    docker.build("${IMAGE_NAME}:${appVersion}")
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID, url: 'https://index.docker.io/v1/') {
                        def appVersion = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
                        docker.image("${IMAGE_NAME}:${appVersion}").push()
                        docker.image("${IMAGE_NAME}:${appVersion}").push("latest")
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def appVersion = sh(script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout", returnStdout: true).trim()
                    sh "kubectl set image deployment/products-api products-api=${IMAGE_NAME}:${appVersion} -n ${K8S_NAMESPACE}"
                }
            }
        }
    }
}
