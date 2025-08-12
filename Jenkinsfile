pipeline {
    agent any

    tools {
        nodejs "Node_24.5.0"
    }

    environment {
        DOCKER_COMPOSE_FILE = "docker-compose.yml"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/tonuser/tonrepo.git'
            }
        }

        stage('Install Backend') {
            steps {
                dir('bibliflow-backend') {
                    sh 'npm install'
                }
            }
        }

        stage('Install Frontend') {
            steps {
                dir('bibliflow-frontend') {
                    sh 'npm install'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('bibliflow-backend') {
                    sh 'npm test'
                }
                dir('bibliflow-frontend') {
                    sh 'npm run test -- --watch=false'
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