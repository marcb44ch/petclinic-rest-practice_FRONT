pipeline {
    agent any

    tools {
        nodejs 'my-node'
    }

    environment {
        SONAR_SERVER_NAME = 'sonar-server'
        CHROME_BIN = '/usr/bin/chromium'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install System Dependencies') {
            steps {
                sh '''
                    echo "=== Instalando Chromium ==="
                    apt-get update
                    apt-get install -y chromium chromium-l10n
                    
                    # Crear symlink si es necesario
                    ln -sf /usr/bin/chromium /usr/bin/chrome || true
                    ln -sf /usr/bin/chromium /usr/bin/google-chrome || true
                    
                    # Verificar instalación
                    which chromium || echo "Chromium no disponible"
                '''
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
                    echo "=== Configurando browser para tests ==="
                    # Configurar browser para Karma
                    export CHROME_BIN=/usr/bin/chromium
                    
                    echo "=== Ejecutando tests ==="
                    npx ng test --watch=false --code-coverage --browsers=ChromeHeadless
                '''
            }
            post {
                always {
                    archiveArtifacts artifacts: 'coverage/**/*', allowEmptyArchive: true
                    sh '''
                        echo "=== Buscando reporte de cobertura del proyecto ==="
                        find . -name "lcov.info" -type f | grep -v node_modules | head -1 | xargs -I {} ls -la {} || echo "No se encontró lcov.info del proyecto"
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