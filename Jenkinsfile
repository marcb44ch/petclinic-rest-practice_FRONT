pipeline {
    agent any

    tools {
        nodejs 'my-node'
    }

    environment {
        SONAR_SERVER_NAME = 'sonar-server'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Build Application') {
            steps {
                sh 'npm run build -- --configuration=production'
            }
        }

        stage('Test with Coverage') {
            steps {
                sh 'npx ng test --watch=false --code-coverage --browsers=ChromeHeadless'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                    sh '''
                        echo "=== Verificando cobertura generada ==="
                        find . -name "lcov.info" -type f | head -1 | xargs -I {} echo "Archivo encontrado: {}" || echo "No se generó lcov.info"
                    '''
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
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    
    post {
        always {
            echo "=== Estado del Pipeline: ${currentBuild.result} ==="
        }
        success {
            echo "✅ Pipeline completado exitosamente!"
        }
        failure {
            echo "❌ Pipeline falló - Revisa los logs"
        }
    }
}