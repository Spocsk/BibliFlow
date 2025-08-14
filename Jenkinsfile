pipeline {
  agent any

  tools {
    nodejs "Node_24.5.0"
  }

  environment {
    // Chemins ABSOLUS vers les fichiers de la racine du workspace
    DOCKER_COMPOSE_FILE = "${WORKSPACE}/compose.yml"
    DOCKER_ENV_FILE     = "${WORKSPACE}/.env"
    CI = "true"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
      }
    }

    // (tes stages Install Backend / Frontend restent identiques)

    // Petit check utile avant build/deploy
    stage('Preflight') {
      steps {
        dir("${WORKSPACE}") {
          sh 'pwd && ls -la'
          sh 'test -f ${DOCKER_ENV_FILE} && echo "OK: .env found" || (echo "ERROR: .env missing" && exit 1)'
          sh 'test -f ${DOCKER_COMPOSE_FILE} && echo "OK: compose.yml found" || (echo "ERROR: compose.yml missing" && exit 1)'
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        // --project-directory force Compose à résoudre les chemins relatifs depuis la racine du repo
        sh 'docker compose --project-directory ${WORKSPACE} --env-file ${DOCKER_ENV_FILE} -f ${DOCKER_COMPOSE_FILE} build'
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker compose --project-directory ${WORKSPACE} --env-file ${DOCKER_ENV_FILE} -f ${DOCKER_COMPOSE_FILE} up -d'
      }
    }
  }
}