pipeline {
  agent any

  tools {
    nodejs "Node_24.5.0"
  }

  environment {
    // compose.yml est à la racine du workspace
    DOCKER_COMPOSE_FILE = "${WORKSPACE}/compose.yml"
    COMPOSE_PROJECT_NAME = "bibliflow-ci-${BUILD_NUMBER}"
    CI = "true"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
      }
    }

    // (tes stages Install Backend / Frontend restent identiques)

    // Récupère le .env depuis les Credentials et l'écrit à la racine du workspace
    stage('Prepare .env (from credentials)') {
      steps {
        withCredentials([file(credentialsId: 'BIBLIFLOW_DOTENV_FILE', variable: 'ENV_FILE')]) {
          sh '''
            cp "$ENV_FILE" "${WORKSPACE}/.env"
            chmod 600 "${WORKSPACE}/.env"
            echo "✓ .env copied to ${WORKSPACE}/.env"
          '''
        }
      }
    }

    // Sanity checks
    stage('Preflight') {
      steps {
        dir("${WORKSPACE}") {
          sh 'pwd && ls -la'
          sh 'test -f ${DOCKER_COMPOSE_FILE} && echo "OK: compose.yml found" || (echo "ERROR: compose.yml missing" && exit 1)'
          sh 'test -f .env && echo "OK: .env present" || (echo "ERROR: .env missing" && exit 1)'
        }
      }
    }

    stage('Cleanup conflicting containers') {
      steps {
        // Remove containers that use fixed container_name in compose.yml (to avoid name conflicts)
        sh 'docker rm -f bibliflow_mongodb bibliflow_postgres bibliflow_backend bibliflow_frontend 2>/dev/null || true'
      }
    }

    stage('Teardown previous CI stack') {
      steps {
        // Clean any previous CI project with the same COMPOSE_PROJECT_NAME
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME} -f ${DOCKER_COMPOSE_FILE} down --remove-orphans || true'
      }
    }

    stage('Build Docker Images') {
      steps {
        // Pas besoin de --env-file ici : les services utilisent env_file: .env
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME} --project-directory ${WORKSPACE} -f ${DOCKER_COMPOSE_FILE} build postgres mongodb backend frontend'
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME} --project-directory ${WORKSPACE} -f ${DOCKER_COMPOSE_FILE} up -d postgres mongodb backend frontend'
      }
    }
  }

  post {
    // Nettoyer le .env éphémère après le job
    always {
      sh 'shred -u "${WORKSPACE}/.env" || rm -f "${WORKSPACE}/.env" || true'
    }
  }
}