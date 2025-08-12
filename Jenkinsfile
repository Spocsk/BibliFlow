pipeline {
    agent any

    tools {
        nodejs "Node_24.5.0"
    }

    environment {
        DOCKER_COMPOSE_FILE = "compose.yml"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
            }
        }

        stage('Install Backend') {
            steps {
                dir('bibliflow-backend') {
                    sh 'npm i'
                }
            }
        }

        stage('Install Frontend') {
            steps {
                dir('bibliflow-frontend') {
                    sh 'npm i'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('bibliflow-backend') {
                    sh 'npm run test'
                }
                dir('bibliflow-frontend') {
                    sh 'npm run test'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                sh "docker compose -f ${DOCKER_COMPOSE_FILE} build"
            }
        }

        stage('Deploy') {
            steps {
                sh "docker compose -f ${DOCKER_COMPOSE_FILE} up -d"
            }
        }
    }
}