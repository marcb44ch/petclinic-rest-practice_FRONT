pipeline {
    agent {
        docker {
            image 'markhobson/node-chrome:latest'  // Imagen con Node + Chrome preinstalado
            args '--user root'  // Ejecutar como root dentro del contenedor
        }
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
                sh '''
                    echo "=== Verificando Chrome ==="
                    which google-chrome || echo "Chrome no disponible"
                    
                    echo "=== Ejecutando tests ==="
                    npx ng test --watch=false --code-coverage --browsers=ChromeHeadless
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                    sh '''
                        echo "=== Verificando cobertura generada ==="
                        if [ -d "coverage" ]; then
                            echo "✅ Directorio coverage encontrado:"
                            ls -la coverage/
                            find coverage -name "lcov.info" -type f | head -1 | xargs -I {} echo "Archivo: {}" || echo "No hay lcov.info en coverage"
                        else
                            echo "❌ No se generó directorio coverage"
                        fi
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
    }
}