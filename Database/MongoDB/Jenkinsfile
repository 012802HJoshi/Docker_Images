pipeline {
    agent any

    environment {
        APP_NAME = "node-app"
        IMAGE_NAME = "node-app-image"
        MONGO_COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Create Environment File') {
            steps{
               script {
                    withCredentials([file(credentialsId: 'ENV_MONGO_DB', variable: 'ENV')]) {
                        sh 'touch .env'
                        sh 'chown jenkins:jenkins .env'
                        sh 'chmod 664 .env'
                        sh 'cp $ENV .env' 
                        sh 'echo "running Docker Container"'
                    }
               }
            }
            
        }

        stage('Docker Info') {
    steps {
        sh 'docker --version'
        script {
            def composeInstalled = sh(
                script: 'docker compose version 2>/dev/null || docker-compose --version 2>/dev/null',
                returnStatus: true
            )
            if (composeInstalled != 0) {
                error("Docker Compose is not installed. Please install it on the Jenkins agent.")
            }
        }
    }
}

        stage('Create Network If Not Exists') {
            steps {
                sh '''
                if ! docker network ls | grep db-network; then
                  docker network create db-network
                fi
                '''
            }
        }

        stage('Start MongoDB') {
            steps {
                sh '''
                docker compose -f ${MONGO_COMPOSE_FILE} up -d
                '''
            }
        }
        }
    }

    post {
        success {
            echo "✅ Deployment Completed Successfully"
        }
        failure {
            echo "❌ Deployment Failed"
        }
    }
