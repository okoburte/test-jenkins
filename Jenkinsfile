pipeline {
  agent any

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
          sudo apt-get update
          DEBIAN_FRONTEND=noninteractive sudo apt-get install -y apache2 curl
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
          sudo mkdir -p "${APACHE_DIR}"
          # copie aussi les fichiers cachés (ex: .htaccess)
          sudo cp -r . "${APACHE_DIR}"
        '''
      }
    }

    stage('Test') {
      steps {
        echo 'Vérification du déploiement...'
        sh 'curl -I http://localhost'
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
      // Nettoyage (laisse Apache installé pour éviter de casser l’agent)
      sh 'sudo rm -rf "${APACHE_DIR:?}"/* || true'
      // Si tu tiens à désinstaller, décommente la ligne suivante :
      sh 'sudo apt-get remove -y apache2 || true'
    }
  }
}
