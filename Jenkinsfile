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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Start Services') {
            steps {
                echo '🚀 Starting DB + App with docker compose...'
                // DB_HOST_PORT can be overridden if 3307 is taken:
                //   e.g. set DB_HOST_PORT=3308 in Jenkins env vars
                sh 'docker compose up -d --build'
            }
        }

        stage('Verify DB & Tables') {
            steps {
                echo '✅ Waiting for MariaDB to be ready...'
                // Give MariaDB a few seconds to initialise
                sh 'sleep 10'
                echo '🔍 Checking that all tables exist...'
                sh '''
                    docker exec calculator-db mariadb -uroot -pTest12 calc_data -e "
                        SHOW TABLES;
                        DESCRIBE calc_results;
                        DESCRIBE subtraction_results;
                        DESCRIBE division_results;
                    "
                '''
            }
        }
    }

    post {
        success {
            echo '🎉 Pipeline completed successfully! DB and tables verified.'
        }
        failure {
            echo '❌ Pipeline failed. Check the logs above.'
            sh 'docker compose logs --tail=50'
        }
        always {
            echo '🧹 Pipeline finished.'
        }
    }
}
