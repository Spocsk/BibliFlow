pipeline {
    agent any
    tools {
        nodejs "Node_24.5.0"
    }
    environment {
        DOCKER_COMPOSE_FILE = "compose.yml"
        CI = "true"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
            }
        }
        // stage('Install Browser') {
        //     steps {
        //         sh '''
        //             # Installation simple de Chromium (disponible sur toutes les architectures)
        //             apt-get update
        //             apt-get install -y chromium xvfb --no-install-recommends
                    
        //             # Vérification
        //             chromium --version
                    
        //             # Variables d'environnement
        //             export CHROME_BIN=/usr/bin/chromium
        //             export PUPPETEER_SKIP_CHROMIUM_DOWNLOAD=true
                    
        //             echo "✓ Navigateur installé: $(chromium --version)"
        //         '''
        //     }
        // }
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
        // stage('Run Tests') {
        //     environment {
        //         CHROME_BIN = '/usr/bin/chromium'
        //         DISPLAY = ':99'
        //     }
        //     steps {
        //         dir('bibliflow-backend') {
        //             sh 'npm run test'
        //         }
        //         dir('bibliflow-frontend') {
        //             sh '''
        //                 # Démarrer Xvfb
        //                 Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
        //                 sleep 2
                        
        //                 # Lancer les tests (Karma va automatiquement utiliser ChromeHeadlessCI)
        //                 npm run test:ci
        //             '''
        //         }
        //     }
        // }

        stage('Build Docker Images') {
            steps {
                dir("${WORKSPACE}") {
                    sh "docker compose -f ${DOCKER_COMPOSE_FILE} build"
                }
            }
        }
        stage('Deploy') {
            steps {
                dir("${WORKSPACE}") {
                    sh "docker compose -f ${DOCKER_COMPOSE_FILE} --env-file .env up -d"
                }
            }
        }
    }
    post {
        always {
            sh 'pkill -f Xvfb || true'
        }
    }
}