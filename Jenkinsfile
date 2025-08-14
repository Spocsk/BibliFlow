pipeline {
    agent any
    tools {
        nodejs "Node_24.5.0"
    }
    environment {
        DOCKER_COMPOSE_FILE = "compose.yml"
        CHROME_BIN = "/usr/bin/chromium"
        PUPPETEER_SKIP_CHROMIUM_DOWNLOAD = "true"
        PUPPETEER_EXECUTABLE_PATH = "/usr/bin/chromium"
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
                    # Update package lists
                    apt-get update
                    
                    # Install Chromium and dependencies for ARM64/AMD64
                    apt-get install -y chromium xvfb dbus-x11 --no-install-recommends
                    
                    # Verify chromium installation
                    chromium --version || echo "Chromium version check failed"
                    which chromium || echo "Chromium not found in PATH"
                    
                    # Create symlinks for compatibility
                    ln -sf /usr/bin/chromium /usr/bin/chromium-browser || true
                    ln -sf /usr/bin/chromium /usr/bin/google-chrome || true
                    
                    # Clean up apt cache
                    rm -rf /var/lib/apt/lists/*
                    
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