pipeline {
    agent any
    environment {
        IMAGE_NAME = "securedev-web"
        CONTAINER_NAME = "securedev-prod"
    }
    stages {
        stage('1. Construccion') {
            steps {
                bat 'git log -n 3 --oneline > trazabilidad_git.txt'
                bat 'docker build -t %IMAGE_NAME%:latest .'
            }
        }
        stage('2. Revision Dependencias y SAST') {
            steps {
                bat 'pip install pip-audit bandit'
                bat 'pip-audit -r requirements.txt -f txt -o reporte_dependencias.txt || exit 0'
                bat 'bandit -r app.py -f txt -o reporte_sast.txt || exit 0'
            }
        }
        stage('3. Despliegue') {
            steps {
                bat 'docker rm -f %CONTAINER_NAME% || exit 0'
                bat 'docker run -d --name %CONTAINER_NAME% -p 5000:5000 %IMAGE_NAME%:latest'
                bat 'ping 127.0.0.1 -n 6 > nul'
            }
        }
        stage('4. Pruebas OWASP ZAP') {
            steps {
                bat 'docker run --rm -v "%WORKSPACE%:/zap/wrk/:rw" ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t "http://host.docker.internal:5000/hello?name=test" -r reporte_zap.html || exit 0'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'trazabilidad_git.txt, reporte_dependencias.txt, reporte_sast.txt, reporte_zap.html', allowEmptyArchive: true
        }
    }
}