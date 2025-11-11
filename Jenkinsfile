pipeline {
    agent any

    tools {
        jdk 'JDK21'
    }

    environment {
        SONARQUBE = 'Dev'     // must match Jenkins config
        DOCKER_IMAGE = 'Dev'
        DOCKER_TAG = "v${BUILD_NUMBER}"
        NEXUS_REPO = 'localhost:8082/devops-docker' // your Nexus Docker repo (no http://)
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
                    // use the configured SonarQube server and scanner tool
                    def scannerHome = tool 'sonar-scanner'
                    withSonarQubeEnv("${SONARQUBE}") {
                        bat """
                            "${scannerHome}\\bin\\sonar-scanner.bat" ^
                                -Dsonar.projectKey=Dev ^
                                -Dsonar.sources=. ^
                                -Dsonar.java.binaries=. ^
                                -Dsonar.host.url=http://localhost:9000
                        """
                    }
                }
            }
        }

        stage('Quality Gate Check') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }

        stage('Push Docker Image to Nexus') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    script {
                        bat """
                            docker login -u ${NEXUS_USER} -p ${NEXUS_PASS} ${NEXUS_REPO}
                            docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${NEXUS_REPO}/${DOCKER_IMAGE}:${DOCKER_TAG}
                            docker push ${NEXUS_REPO}/${DOCKER_IMAGE}:${DOCKER_TAG}
                        """
                    }
                }
            }
        }
    }
}
