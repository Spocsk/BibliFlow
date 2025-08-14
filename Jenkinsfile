pipeline {
  agent any

  tools {
    nodejs "Node_24.5.0"
  }

  environment {
    // Chemin absolu du compose à la racine du workspace
    DOCKER_COMPOSE_FILE = "${WORKSPACE}/compose.yml"
    CI = "true"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
      }
    }

    // (tes stages Install Backend / Frontend restent identiques)

    // Vérifie compose.yml et la présence du credential .env
    stage('Preflight') {
      steps {
        dir("${WORKSPACE}") {
          sh 'pwd && ls -la'
          sh 'test -f ${DOCKER_COMPOSE_FILE} && echo "OK: compose.yml found" || (echo "ERROR: compose.yml missing" && exit 1)'
        }
        withCredentials([file(credentialsId: 'BIBLIFLOW_DOTENV_FILE', variable: 'ENV_FILE')]) {
          sh 'echo "Using env file at: $ENV_FILE" && test -s "$ENV_FILE" && echo "OK: secret .env provided" || (echo "ERROR: secret .env is empty/missing" && exit 1)'
        }
      }
    }

    stage('Build Docker Images') {
      steps {
        withCredentials([file(credentialsId: 'BIBLIFLOW_DOTENV_FILE', variable: 'ENV_FILE')]) {
          sh 'docker compose --project-directory ${WORKSPACE} --env-file "$ENV_FILE" -f ${DOCKER_COMPOSE_FILE} build'
        }
      }
    }

    stage('Deploy') {
      steps {
        withCredentials([file(credentialsId: 'BIBLIFLOW_DOTENV_FILE', variable: 'ENV_FILE')]) {
          sh 'docker compose --project-directory ${WORKSPACE} --env-file "$ENV_FILE" -f ${DOCKER_COMPOSE_FILE} up -d'
        }
      }
    }
  }
}