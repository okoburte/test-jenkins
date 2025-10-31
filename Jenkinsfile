pipeline {
  agent any

  environment {
    REPO_URL  = 'https://github.com/okoburte/test-jenkins.git'
    APACHE_DIR = '/var/www/html'
    SITE_DIR  = 'site-local'   // fallback sans root
    PID_FILE  = '.server.pid'
    USE_APACHE = ''            // sera "yes" si root
  }

  stages {
    stage('Dépendances (conditionnel)') {
      steps {
        echo 'Préparation de l’environnement...'
        sh '''
          set -eu
          if [ "$(id -u)" -eq 0 ]; then
            export DEBIAN_FRONTEND=noninteractive
            apt-get update
            apt-get install -y --no-install-recommends apache2 curl ca-certificates
            echo "yes" > .use_apache
          else
            echo "no" > .use_apache
          fi
        '''
      }
    }

    stage('Checkout') {
      steps {
        echo 'Récupération du code source...'
        git "${REPO_URL}"
      }
    }

    stage('Déploiement') {
      steps {
        sh '''
          set -eu
          USE_APACHE=$(cat .use_apache || echo no)
          if [ "$USE_APACHE" = "yes" ]; then
            echo "Déploiement Apache..."
            rm -rf "${APACHE_DIR:?}"/*
            cp -R . "${APACHE_DIR}"
            apache2ctl -k start || true
            echo "yes" > .use_apache
          else
            echo "Déploiement local (python http.server)..."
            rm -rf "${SITE_DIR}" && mkdir -p "${SITE_DIR}"
            cp -R . "${SITE_DIR}"
            (cd "${SITE_DIR}" && (python3 -m http.server 8000 || python -m SimpleHTTPServer 8000) >/dev/null 2>&1 & echo $! > "../${PID_FILE}")
            echo "no" > .use_apache
            sleep 1
          fi
        '''
      }
    }

    stage('Test') {
      steps {
        echo 'Vérification du déploiement...'
        sh '''
          set -eu
          USE_APACHE=$(cat .use_apache || echo no)
          if [ "$USE_APACHE" = "yes" ]; then
            curl -I http://localhost | head -n1
          else
            curl -I http://127.0.0.1:8000 | head -n1
          fi
        '''
      }
    }

    stage('Archivage') {
      steps {
        sh 'test -f index.html && echo OK || true'
        archiveArtifacts artifacts: 'index.html, **/index.html', fingerprint: true, allowEmptyArchive: true
      }
    }
  }

  post {
    success {
      echo '✅ Déploiement réussi.'
    }
    failure {
      echo '❌ Échec du déploiement.'
    }
    always {
      echo '🧹 Nettoyage...'
      sh '''
        set +e
        # stop Apache si lancé
        apache2ctl -k stop 2>/dev/null || true
        # tue le serveur python si lancé
        [ -f "${PID_FILE}" ] && kill "$(cat ${PID_FILE})" 2>/dev/null || true
      '''
    }
  }
}
