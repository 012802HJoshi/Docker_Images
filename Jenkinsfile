pipeline {
    agent any

    environment {
        APP_NAME = "node-app"
        IMAGE_NAME = "node-app-image"
        MONGO_COMPOSE_FILE = "docker-compose.yml"
        DOCKER_NETWORK = "db-network"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Create Environment File') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'ENV_MONGO_DB', variable: 'ENV')]) {
                        sh '''
                            echo "Creating .env file..."
                            touch .env
                            chown jenkins:jenkins .env
                            chmod 664 .env
                            cp $ENV .env
                            echo "✅ .env file created"
                        '''
                    }
                }
            }
        }

        stage('Docker Info') {
            steps {
                sh 'docker --version'
                script {
                    def composeInstalled = sh(
                        script: 'docker compose version >/dev/null 2>&1',
                        returnStatus: true
                    )

                    if (composeInstalled != 0) {
                        error("❌ Docker Compose is not installed on this Jenkins agent.")
                    } else {
                        echo "✅ Docker Compose is available"
                    }
                }
            }
        }

        stage('Create Network If Not Exists') {
            steps {
                sh '''
                    if ! docker network ls | grep -q ${DOCKER_NETWORK}; then
                        echo "Creating Docker network: ${DOCKER_NETWORK}"
                        docker network create ${DOCKER_NETWORK}
                    else
                        echo "Docker network already exists: ${DOCKER_NETWORK}"
                    fi
                '''
            }
        }

        stage('Start MongoDB') {
            steps {
                sh '''
                    echo "Starting MongoDB using Docker Compose..."
                    docker compose -f ${MONGO_COMPOSE_FILE} up -d
                    echo "✅ MongoDB deployment command executed"
                '''
            }
        }

        stage('Verify MongoDB') {
            steps {
                sh '''
                    echo "Verifying MongoDB container..."
                    docker ps | grep mongodb
                '''
            }
        }
    }

    post {
        success {
            echo "✅✅✅ MongoDB Pipeline Completed Successfully ✅✅✅"
        }
        failure {
            echo "❌❌❌ MongoDB Pipeline Failed ❌❌❌"
        }
    }
}
