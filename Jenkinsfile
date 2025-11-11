pipeline {
    agent any

    tools {
        jdk 'JDK21'
    }

    environment {
        SONARQUBE = 'SonarQube'
        DOCKER_IMAGE = 'devops-revision'
        DOCKER_TAG = "v${BUILD_NUMBER}"
        NEXUS_REPO = 'http://localhost:8081/'
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
                    withSonarQubeEnv("${SONARQUBE}") {
                        bat """
                            sonar-scanner ^
                                -Dsonar.projectKey=devops-revision ^
                                -Dsonar.sources=. ^
                                -Dsonar.java.binaries=. ^
                                -Dsonar.host.url=http://localhost:9000 ^
                                -Dsonar.login=sqp_ce843eb188f32ba09cc211ec1c47f0d7929cea19
                        """
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
