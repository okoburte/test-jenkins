pipeline {
  agent {
    docker {
      image 'debian:bookworm-slim'
      args  '-u root:root'   // on exécute en root pour apt et /var/www/html
    }
  }

  environment {
    APACHE_DIR = '/var/www/html'
    REPO_URL   = 'https://github.com/okoburte/test-jenkins.git'
  }

  stages {
    stage('Dependances') {
      steps {
        echo 'Installation des dépendances...'
        sh '''
          set -eu
          apt-get update
          DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends apache2 curl ca-certificates
        '''
      }
    }

    stage('Checkout') {
      steps {
        echo 'Récupération du code source...'
        git "${REPO_URL}"
      }
    }

    stage('Deploy') {
      steps {
        echo 'Déploiement des fichiers...'
        sh '''
          set -eu
          mkdir -p "${APACHE_DIR}"
          rm -rf "${APACHE_DIR:?}"/*
          # copie y compris fichiers cachés (.htaccess, etc.)
          cp -R . "${APACHE_DIR}"
          # démarre Apache dans le conteneur (pas de systemd, on utilise apache2ctl)
          apache2ctl -k start
        '''
      }
    }

    stage('Test') {
      steps {
        echo 'Vérification du déploiement...'
        sh 'curl -I http://localhost | head -n1'
      }
    }
  }

  post {
    success {
      echo 'Déploiement réussi.'
    }
    failure {
      echo 'Échec du déploiement.'
    }
    always {
      // nettoyage dans le conteneur; on stoppe Apache proprement
      sh '''
        set -e
        apache2ctl -k stop || true
      '''
    }
  }
}
