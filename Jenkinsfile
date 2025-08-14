pipeline {
    agent any
    tools {
        nodejs "Node_24.5.0"
    }
    environment {
        DOCKER_COMPOSE_FILE = "compose.yml"
        CHROME_BIN = "/usr/bin/chromium"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
            }
        }
        stage('Install System Dependencies') {
            steps {
                sh '''
                    apt-get update
                    apt-get install -y chromium chromium-driver xvfb
                    
                    # Verify chromium installation
                    which chromium || echo "Chromium not found in PATH"
                    ls -la /usr/bin/chromium* || echo "No chromium binaries found"
                    
                    # Create symlink if needed
                    if [ ! -f /usr/bin/chromium-browser ] && [ -f /usr/bin/chromium ]; then
                        ln -sf /usr/bin/chromium /usr/bin/chromium-browser
                    fi
                    
                    echo "CHROME_BIN will be: $CHROME_BIN"
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
                    sh '''
                        # Verify Chrome is available
                        echo "CHROME_BIN is set to: $CHROME_BIN"
                        ls -la $CHROME_BIN || echo "Chrome binary not found at $CHROME_BIN"
                        
                        # Run tests with explicit Chrome path
                        CHROME_BIN=$CHROME_BIN npm run test:ci -- --browsers=ChromeHeadlessNoSandbox
                    '''
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