pipeline {
    agent any

    tools {
        nodejs "Node_24.5.0"
    }

    environment {
        DOCKER_COMPOSE_FILE = "compose.yml"
        CHROME_BIN = "/usr/bin/chromium-browser"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
            }
        }

        stage('Install Chromium') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y chromium-browser
                    echo "CHROME_BIN=$(which chromium-browser)"
                    which chromium-browser
                '''
            }
        }

        stage('Install Backend') {
            steps {
                dir('bibliflow-backend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Install Frontend') {
            steps {
                dir('bibliflow-frontend') {
                    sh 'npm ci'
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('bibliflow-backend') {
                    sh 'npm run test'
                }
                dir('bibliflow-frontend') {
                    sh 'npm run test:ci -- --browsers=ChromeHeadlessNoSandbox'
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