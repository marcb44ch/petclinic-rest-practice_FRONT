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
                    # Instalar las dependencias CORRECTAS de Karma
                    npm install --save-dev karma-chrome-launcher karma-coverage
                    # Verificar que están instaladas
                    npm list karma-chrome-launcher karma-coverage
                '''
            }
        }

        stage('Test with Coverage') {
            steps {
                script {
                    try {
                        // Construir la aplicación primero
                        sh 'npm run build -- --prod'
                        
                        // Ejecutar tests
                        sh 'npm run test -- --watch=false --code-coverage --browsers=ChromeHeadless'
                    } catch (error) {
                        echo "Tests fallaron pero continuamos: ${error}"
                        // Continuar para intentar generar cobertura
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
                script {
                    // Verificar que existe el reporte de cobertura
                    sh 'find . -name "lcov.info" -type f | head -1 || echo "No se encontró lcov.info"'
                }
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
