pipeline {
  agent any

  options {
    disableConcurrentBuilds()
  }

  tools {
    nodejs "Node_24.5.0"
  }

  environment {
    // === Deploy ===
    DOCKER_COMPOSE_DEPLOY_BASE = "${WORKSPACE}/compose.yml"
    DOCKER_COMPOSE_DEPLOY_OVR  = "${WORKSPACE}/compose.ci.deploy.yml" // optionnel
    COMPOSE_PROJECT_NAME_DEPLOY = "bibliflow"

    CI = "true"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
      }
    }

    stage('Preflight') {
      steps {
        dir("${WORKSPACE}") {
          sh 'pwd && ls -la'
          sh 'test -f ${DOCKER_COMPOSE_DEPLOY_BASE} && echo "OK: compose.yml found" || (echo "ERROR: compose.yml missing" && exit 1)'
          // l'override de déploiement est optionnel
          sh '[ -f ${DOCKER_COMPOSE_DEPLOY_OVR} ] && echo "compose.ci.deploy.yml present (will be used)" || echo "compose.ci.deploy.yml not present (will be skipped)"'
        }
      }
    }

    // Préparer .env depuis Credentials (Secret file)
    stage('Prepare .env (from credentials)') {
      steps {
        withCredentials([file(credentialsId: 'BIBLIFLOW_DOTENV_FILE', variable: 'ENV_FILE')]) {
          sh 'cp "$ENV_FILE" "${WORKSPACE}/.env" && chmod 600 "${WORKSPACE}/.env" && echo "✓ .env copied"'
        }
      }
    }

    // Construire la liste des fichiers -f à utiliser (override optionnel)
    stage('Assemble deploy compose files') {
      steps {
        sh '''
          DEPLOY_FILES="-f ${DOCKER_COMPOSE_DEPLOY_BASE}"
          if [ -f "${DOCKER_COMPOSE_DEPLOY_OVR}" ]; then
            DEPLOY_FILES="$DEPLOY_FILES -f ${DOCKER_COMPOSE_DEPLOY_OVR}"
          fi
          echo "$DEPLOY_FILES" > .deploy_files
          echo "Using compose files: $(cat .deploy_files)"
        '''
      }
    }

    // Down → Build → Up
    stage('Deploy - Down') {
      steps {
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME_DEPLOY} --project-directory ${WORKSPACE} $(cat .deploy_files) down --remove-orphans || true'
      }
    }

    stage('Deploy - Build') {
      steps {
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME_DEPLOY} --project-directory ${WORKSPACE} $(cat .deploy_files) build postgres mongodb backend frontend'
      }
    }

    stage('Deploy - Up') {
      steps {
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME_DEPLOY} --project-directory ${WORKSPACE} $(cat .deploy_files) up -d postgres mongodb backend frontend'
      }
    }
  }

  post {
    always {
      // Nettoyage des fichiers sensibles/temporaires
      sh 'shred -u "${WORKSPACE}/.env" || rm -f "${WORKSPACE}/.env" || true'
      sh 'rm -f .deploy_files || true'
    }
  }
}