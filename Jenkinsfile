pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'nhatphuoc/spring-petclinic'
        REGISTRY_CREDENTIALS = 'devOps_project02'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Get Commit ID') {
            steps {
                script {
                    COMMIT_ID = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Commit ID: ${COMMIT_ID}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE}:${COMMIT_ID}")
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', REGISTRY_CREDENTIALS) {
                        docker.image("${DOCKER_IMAGE}:${COMMIT_ID}").push()
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pushed ${DOCKER_IMAGE}:${COMMIT_ID} to Docker Hub successfully"
        }
        failure {
            echo "Build or push failed"
        }
    }
}
