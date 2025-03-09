pipeline {
    agent any
     environment {
            // Define Docker Hub credentials ID
            DOCKERHUB_CREDENTIALS_ID = 'Docker-hub-credentials'
            // Define Docker Hub repository name
            DOCKERHUB_REPO = 'topskari/docker-test'
            // Define Docker image tag
            DOCKER_IMAGE_TAG = 'latest_v1'
        }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/topskari/docker-test'
            }
        }
        stage('Build') {
            steps {
                bat 'mvn clean install'
            }
        }
        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }
        stage('Code Coverage') {
            steps {
                bat 'mvn jacoco:report'
            }
        }
        stage('Publish Test Results') {
            steps {
                junit '**/target/surefire-reports/*.xml'
            }
        }

         stage('Build Docker Image') {
                    steps {
                        // Build Docker image
                        script {
                            docker.build("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}")
                        }
                    }
                }
                stage('Push Docker Image to Docker Hub') {
                    steps {
                        // Push Docker image to Docker Hub
                        script {
                            docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS_ID) {
                                docker.image("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}").push()
                            }
                        }
                    }
                }
    }
    post {
                always {
                    recordCoverage(
                        tools: [[parser: 'JACOCO', pattern: 'Brainboost/target/site/jacoco/jacoco.xml']],
                        checksAnnotationScope: 'ALL_LINES',
                        enabledForFailure: true,
                        failOnError: true,
                        sourceCodeRetention: 'LAST_BUILD'
                    )
                }
            }
        }
}