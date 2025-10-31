pipeline {
    agent any

        environment {
            APACHE_DIR = '/var/www/html'
            REPO_URL = 'https://github.com/okoburte/test-jenkins.git'
        }

    stages {
        stage('Dependances') {
            steps {
                // Installation des dépendances (apache2)
                echo 'Installation des dépendances...'
                sh 'apt-get update'
                sh 'apt-get install -y apache2'
            }
        }
        stage('Checkout') {
            steps {
                // Récupération du code
                echo 'Récupération du code source...'
                git "${REPO_URL}"
            
            }
        }
        stage('Deploy') {
            steps {
                // Copie des fichiers vers le serveur web (/var/www/html/)
                echo 'Déploiement des fichiers...'
                sh "cp -r * ${APACHE_DIR}"
            }
        }
        stage('Test') {
            steps {
                // Vérification du déploiement
                echo 'Vérification du déploiement...'
                sh "curl -I http://localhost"
            }
        }
    }
    post {
        success {
            // Message de succès
            echo 'Déploiement réussi.'
        }
        failure {
            // Message d'échec
            echo 'Échec du déploiement.'
        }
        always {
            // Nettoyage
            // Supprimer les fichiers copiés dans /var/www/html
            sh "rm -rf ${APACHE_DIR}/*"
            // Desinstaller apache2
            sh 'apt-get remove -y apache2'
        }
    }
}
