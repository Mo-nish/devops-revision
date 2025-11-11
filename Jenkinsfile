pipeline {
    agent any

    tools {
        jdk 'JDK21'
    }

    environment {
        SONARQUBE = 'SonarQube'
        DOCKER_IMAGE = 'devops-revision'
        DOCKER_TAG = "v${BUILD_NUMBER}"
        NEXUS_REPO = 'nexus-repo-url'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Mo-nish/devops-revision.git'
            }
        }

        stage('Code Quality - SonarQube Scan') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube') {
                        bat "sonar-scanner -Dsonar.projectKey=devops-revision -Dsonar.sources=."
                    }
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t %DOCKER_IMAGE%:%DOCKER_TAG% ."
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            steps {
                script {
                    // Replace with your Nexus Docker repo login
                    bat "docker login -u <username> -p <password> <nexus-url>"
                    bat "docker tag %DOCKER_IMAGE%:%DOCKER_TAG% <nexus-url>/%DOCKER_IMAGE%:%DOCKER_TAG%"
                    bat "docker push <nexus-url>/%DOCKER_IMAGE%:%DOCKER_TAG%"
                }
            }
        }
    }
}
