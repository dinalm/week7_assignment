pipeline {
    agent any

    environment {
        IMAGE_NAME = "dinal1999/week7_assignment"
        IMAGE_TAG  = "latest"
    }

    stages {

        stage('Checkout') {
            steps {
                echo '📥 Cloning repository...'
                checkout scm
            }
        }

        stage('Build JAR') {
            steps {
                echo '🔨 Building fat JAR with Maven...'
                bat 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                bat "docker build -t %IMAGE_NAME%:%IMAGE_TAG% ."
            }
        }

        stage('Push Docker Image') {
            steps {
                echo '📤 Pushing image to Docker Hub...'
                bat "docker push %IMAGE_NAME%:%IMAGE_TAG%"
            }
        }

        stage('Start Services') {
            steps {
                echo '🚀 Starting DB container with docker compose...'
                // Only start the db service — the app needs a display and will crash headless
                bat 'docker compose up -d db'
            }
        }

        stage('Verify DB & Tables') {
            steps {
                echo '⏳ Waiting 30s for MariaDB to fully initialise...'
                // Use PowerShell sleep — "timeout /t" fails when stdin is redirected in Jenkins
                bat 'powershell -Command "Start-Sleep -Seconds 30"'
                echo '🔍 Checking that all tables exist...'
                bat "docker exec calculator-db mariadb -uroot -pTest12 calc_data -e \"SHOW TABLES; DESCRIBE calc_results; DESCRIBE subtraction_results; DESCRIBE division_results;\""
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completed successfully! DB and tables verified.'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs above.'
            bat 'docker compose logs --tail=50'
        }
        always {
            echo '🧹 Pipeline finished.'
        }
    }
}
