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
                sh '''
                    npm install
                    npm list karma-chrome-launcher karma-coverage
                '''
            }
        }

        stage('Build Application') {
            steps {
                sh '''
                    # Verificar qué scripts de build están disponibles
                    npm run | grep build || echo "No se encontraron scripts de build"
                    
                    # Intentar diferentes opciones
                    echo "Intentando build con --configuration=production..."
                    npm run build -- --configuration=production || 
                    (echo "Falló --configuration=production, intentando --prod..." && 
                    npm run build -- --prod) || 
                    (echo "Falló --prod, intentando build simple..." && 
                    npm run build) || 
                    echo "Todos los métodos de build fallaron"
                '''
            }
        }

        stage('Test with Coverage') {
            steps {
                sh '''
                    # Ejecutar tests con cobertura
                    npx ng test --watch=false --code-coverage --browsers=ChromeHeadless
                '''
            }
            post {
                always {
                    // Solo archivar artefactos, sin publishHTML
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                    // Verificar qué se generó
                    sh '''
                        echo "=== Contenido del directorio coverage ==="
                        ls -la coverage/ || echo "No hay directorio coverage"
                        find . -name "lcov.info" -type f | head -1 | xargs -I {} cat {} | head -5 || echo "No se encontró lcov.info"
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    // Verificar que existen los archivos necesarios
                    sh '''
                        echo "=== Verificando archivos para SonarQube ==="
                        find . -name "lcov.info" -type f | head -1 | xargs -I {} echo "Archivo encontrado: {}" || echo "lcov.info no encontrado"
                        find . -name "coverage" -type d | head -1 | xargs -I {} ls -la {} || echo "Directorio coverage no encontrado"
                    '''
                }
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
            echo "=== Pipeline completado ==="
            // Opcional: limpiar workspace si es necesario
            // cleanWs()
        }
        success {
            echo "✅ Pipeline ejecutado exitosamente"
        }
        failure {
            echo "❌ Pipeline falló - revisa los logs para más detalles"
        }
    }
}