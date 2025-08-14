pipeline {
  agent any

  options {
    disableConcurrentBuilds()
  }

  tools {
    nodejs "Node_24.5.0"
  }

  environment {
    DOCKER_COMPOSE_FILE = "${WORKSPACE}/compose.yml"   // compose à la racine
    COMPOSE_PROJECT_NAME = "bibliflow"                 // projet fixe, idempotent
    CI = "true"
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/Spocsk/BibliFlow'
      }
    }

    // Si tu utilises le credential Secret file pour .env :
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

    stage('Preflight') {
      steps {
        dir("${WORKSPACE}") {
          sh 'pwd && ls -la'
          sh 'test -f ${DOCKER_COMPOSE_FILE} && echo "OK: compose.yml found" || (echo "ERROR: compose.yml missing" && exit 1)'
          sh 'test -f .env && echo "OK: .env present" || (echo "ERROR: .env missing" && exit 1)'
        }
      }
    }

    // 1) DOWN — proprement, sans supprimer les volumes (on garde les données)
    stage('Teardown stack') {
      steps {
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME} --project-directory ${WORKSPACE} -f ${DOCKER_COMPOSE_FILE} down --remove-orphans || true'
      }
    }

    // 2) BUILD — rebuild des images (tous les services dont on a besoin)
    stage('Build images') {
      steps {
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME} --project-directory ${WORKSPACE} -f ${DOCKER_COMPOSE_FILE} build postgres mongodb backend frontend'
      }
    }

    // 3) UP — relancer les services (on évite de lancer le service jenkins du compose)
    stage('Deploy stack') {
      steps {
        sh 'docker compose -p ${COMPOSE_PROJECT_NAME} --project-directory ${WORKSPACE} -f ${DOCKER_COMPOSE_FILE} up -d postgres mongodb backend frontend'
      }
    }
  }

  post {
    always {
      // Nettoie l’éventuel .env écrit depuis le credential
      sh 'shred -u "${WORKSPACE}/.env" || rm -f "${WORKSPACE}/.env" || true'
    }
  }
}