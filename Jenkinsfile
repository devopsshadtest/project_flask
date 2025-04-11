pipeline {
    agent any
    environment {
        GITHUB_REPO = 'https://github.com/devopsshadtest/project_flask.git'
        BRANCH = 'Development'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH}", url: "${GITHUB_REPO}"
            }
        }
        stage('Stop Old Containers') {
            steps {
                script {
                    // Перевіряємо, чи є запущені контейнери з docker-compose.yml
                    def running = sh(script: 'docker-compose ps -q', returnStdout: true).trim()
                    if (running) {
                        echo "Stopping old containers..."
                        sh 'docker-compose down'
                    } else {
                        echo "No old containers running."
                    }
                }
            }
        }
        stage('Build App and Deploy with Docker Compose') {
            steps {
                sh 'docker-compose up -d --build'
                sh 'sleep 5'  // Чекаємо, поки сервіси запустяться
            }
        }
        stage('Test Application') {
            steps {
                script {
                    def appContainer = sh(script: 'docker-compose ps -q app', returnStdout: true).trim()
                    if (!appContainer) {
                        error "App container is not running!"
                    }
                    sh "docker exec ${appContainer} pytest /app/tests/test_api.py -v"
                }
            }
        }
        stage('Restart App Container') {
            steps {
                sh 'docker-compose restart app'
            }
        }
        stage('Backup Images') {
            steps {
                //зберігаємо образ app
                sh 'docker save -o flask-app-backup.tar ${JOB_NAME}_app'
                archiveArtifacts artifacts: 'flask-app-backup.tar', allowEmptyArchive: true
            }
        }
    }
    post {
        always {
            // Виводимо логи, але не зупиняємо контейнери
            sh 'docker-compose logs'
        }
        success {
            echo 'Pipeline completed successfully! Containers are running.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details. Containers may still be running.'
        }
    }
}