pipeline {
    agent any

    tools {
        nodejs "NodeJS 18"
    }

    environment {
        NODE_ENV = "development"
    }

    stages {
        stage('Checkout') {
            steps {
                // Récupère le code du dépôt lié au job Jenkins
                checkout scm
            }
        }
        stage('Install') {
            steps {
                echo "Node version:"
                sh 'node -v'
                echo "NPM version:"
                sh 'npm -v'
                echo "Installation des dépendances..."
                sh 'npm install'
            }
        }
        stage('Build') {
            when {
                expression {
                    // Ne lance build que si le script build est défini dans package.json
                    def packageJson = readJSON file: 'package.json'
                    return packageJson.scripts?.build != null
                }
            }
            steps {
                echo "Build du projet..."
                sh 'npm run build'
            }
        }
        stage('Test') {
            steps {
                echo "Lancement des tests..."
                sh 'npm test'
            }
            post {
                always {
                    junit '**/test-results/*.xml' // si tu génères des rapports JUnit, sinon à adapter
                }
            }
        }
    }

    post {
        success {
            echo 'Build terminée avec succès !'
        }
        failure {
            echo 'Build échouée, vérifie les logs.'
        }
        always {
            cleanWs() // nettoyage de l'espace de travail
        }
    }
}
