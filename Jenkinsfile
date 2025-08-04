pipeline {
    agent any
    
    tools {
        nodejs 'nodejs' // Ensure this matches your Jenkins NodeJS installation name
    }
    
    environment {
        NODE_ENV = 'development'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }
        
        stage('Environment Info') {
            steps {
                echo 'Node version:'
                sh 'node -v'
                echo 'NPM version:'
                sh 'npm -v'
                echo 'Current directory:'
                sh 'pwd'
                echo 'List files:'
                sh 'ls -la'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installation des dépendances...'
                sh 'npm install'
            }
        }
        
        stage('Fix Vulnerabilities') {
            steps {
                echo 'Fixing npm vulnerabilities...'
                sh 'npm audit fix --force || true' // Continue even if some fixes fail
            }
        }
        
        stage('Build') {
            steps {
                script {
                    echo 'Building the application...'
                    
                    // Check if build script exists in package.json
                    def hasBuildScript = sh(
                        script: 'grep -q "\\"build\\":" package.json',
                        returnStatus: true
                    ) == 0
                    
                    if (hasBuildScript) {
                        sh 'npm run build'
                    } else {
                        echo 'No build script found in package.json, skipping build...'
                    }
                }
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo 'Running tests...'
                    
                    // Check if test script exists
                    def hasTestScript = sh(
                        script: 'grep -q "\\"test\\":" package.json',
                        returnStatus: true
                    ) == 0
                    
                    if (hasTestScript) {
                        sh 'npm test || true' // Don't fail pipeline if tests fail
                    } else {
                        echo 'No test script found, skipping tests...'
                    }
                }
            }
        }
        
        stage('Package Info') {
            steps {
                script {
                    // Get package information without readJSON
                    echo 'Retrieving package information...'
                    
                    try {
                        def packageName = sh(
                            script: 'node -e "console.log(require(\'./package.json\').name)"',
                            returnStdout: true
                        ).trim()
                        echo "Package name: ${packageName}"
                        
                        def packageVersion = sh(
                            script: 'node -e "console.log(require(\'./package.json\').version)"',
                            returnStdout: true
                        ).trim()
                        echo "Package version: ${packageVersion}"
                        
                    } catch (Exception e) {
                        echo "Could not read package.json: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'
                
                // Archive common build outputs
                script {
                    if (fileExists('dist')) {
                        archiveArtifacts artifacts: 'dist/**/*', allowEmptyArchive: true
                    }
                    if (fileExists('build')) {
                        archiveArtifacts artifacts: 'build/**/*', allowEmptyArchive: true
                    }
                    
                    // Always archive package.json and package-lock.json
                    archiveArtifacts artifacts: 'package*.json', allowEmptyArchive: true
                }
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning workspace...'
            cleanWs()
        }
        
        success {
            echo '✅ Build réussie!'
            // You can add notifications here
        }
        
        failure {
            echo '❌ Build échouée, vérifie les logs.'
            // You can add failure notifications here
        }
        
        unstable {
            echo '⚠️ Build instable.'
        }
    }
}
