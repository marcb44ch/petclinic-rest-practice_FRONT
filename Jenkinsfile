pipeline {
    agent any

    tools {
        nodejs 'my-node'
        'hudson.plugins.sonar.SonarRunnerInstallation' 'my-sonar-scanner'
    }

    environment {
        SONAR_SERVER_NAME = 'sonar-server'
    }

    stages {
        // 1. Checkout [cite: 62]
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        // 2. Instal·lació i Tests amb Cobertura [cite: 63]
        stage('Install & Test') {
            steps {
                sh 'npm ci'
                
                sh 'npm run test -- --watch=false --code-coverage'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                }
            }
        }

        // 3. SonarQube Scan [cite: 64]
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONAR_SERVER_NAME) {
                    sh 'sonar-scanner'
                }
            }
        }
        
        // 4. Quality Gate (Recomanat també pel frontend)
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
