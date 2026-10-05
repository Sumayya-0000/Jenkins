pipeline {
    agent any

    environment {
        IMAGE_NAME = "sumayyasadaf/registration-form"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat "docker build -t %IMAGE_NAME%:%BUILD_NUMBER% ."
                bat "docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest"
            }
        }

        stage('Docker Login') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    bat "docker login -u %DOCKER_USERNAME% -p %DOCKER_PASSWORD%"
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat "docker push %IMAGE_NAME%:%BUILD_NUMBER%"
                bat "docker push %IMAGE_NAME%:latest"
            }
        }
    }
}