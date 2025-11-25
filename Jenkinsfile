pipeline {
    agent any

    tools {
        nodejs 'my-node'
    }

    environment {
        SONAR_SERVER_NAME = 'sonar-server'
        CHROME_BIN = '/usr/bin/chromium-browser'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    npm ci
                    # Instalar las dependencias faltantes de Karma
                    npm install --save-dev karma-chrome-headless karma-coverage
                '''
            }
        }

        stage('Test with Coverage') {
            steps {
                script {
                    try {
                        // Primero verificar que las dependencias están instaladas
                        sh 'npm list karma-chrome-headless karma-coverage'
                        
                        // Ejecutar tests
                        sh 'npm run test -- --watch=false --code-coverage --browsers=ChromeHeadless'
                    } catch (error) {
                        echo "Tests fallaron: ${error}"
                        // Continuar para generar reporte de cobertura
                    }
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'coverage',
                        reportFiles: 'index.html',
                        reportName: 'Coverage Report'
                    ])
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONAR_SERVER_NAME) {
                    sh 'sonar-scanner'
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
