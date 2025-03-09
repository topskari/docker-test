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
                script {
                    bat "docker build -t ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG} ."
                }
            }
        }
        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: DOCKERHUB_CREDENTIALS_ID, passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        bat "docker login -u %DOCKERHUB_USERNAME% -p %DOCKERHUB_PASSWORD%"
                        bat "docker push ${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}"
                    }
                }
            }
        }
    }
    post {
        always {
            recordCoverage(
                tools: [[parser: 'JACOCO', pattern: '**/target/site/jacoco/jacoco.xml']],
                checksAnnotationScope: 'ALL_LINES',
                enabledForFailure: true,
                failOnError: true,
                sourceCodeRetention: 'LAST_BUILD'
            )
        }
    }
}