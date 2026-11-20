pipeline {
    agent any
    environment {
        DOCKER_BIN = "C:/Users/ferna/AppData/Local/Programs/DockerDesktop/resources/bin/docker.exe"
        IMAGE_NAME = "securedev-web"
        CONTAINER_NAME = "securedev-prod"
    }
    stages {
        stage('1. Construccion') {
            steps {
                bat 'git log -n 3 --oneline > trazabilidad_git.txt'
                bat '"%DOCKER_BIN%" build -t %IMAGE_NAME%:latest .'
            }
        }
        stage('2. Revision Dependencias y SAST') {
            steps {
                bat '"%DOCKER_BIN%" run --rm -v "%WORKSPACE%:/app" -w /app python:3.11-slim sh -c "pip install -q pip-audit bandit && (pip-audit -r requirements.txt -f txt -o reporte_dependencias.txt || true) && (bandit -r app.py -f txt -o reporte_sast.txt || true)"'
            }
        }
        stage('3. Despliegue') {
            steps {
                bat '"%DOCKER_BIN%" rm -f %CONTAINER_NAME% || exit 0'
                bat '"%DOCKER_BIN%" run -d --name %CONTAINER_NAME% -p 5000:5000 %IMAGE_NAME%:latest'
                bat 'ping 127.0.0.1 -n 6 > nul'
            }
        }
        stage('4. Pruebas OWASP ZAP') {
            steps {
                bat '"%DOCKER_BIN%" run --rm -v "%WORKSPACE%:/zap/wrk/:rw" ghcr.io/zaproxy/zaproxy:stable zap-baseline.py -t "http://host.docker.internal:5000/hello?name=test" -r reporte_zap.html || exit 0'
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'trazabilidad_git.txt, reporte_dependencias.txt, reporte_sast.txt, reporte_zap.html', allowEmptyArchive: true
        }
    }
}