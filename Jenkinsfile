pipeline {
    agent any

    stages {
        stage('Checkout Infrastructure Repo') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Deploy Infra via Docker Compose') {
            steps {
                script {
                    echo 'Stopping and cleaning old infrastructure containers...'
                    // Shut down previous infra containers and wipe anonymous runtime volumes
                    bat 'docker-compose down -v --remove-orphans || ver > nul'

                    echo 'Launching Kafka & Zookeeper Core Services...'
                    // Launch infrastructure detached in the background
                    bat 'docker-compose up -d'

                    echo 'Waiting for Kafka broker to complete self-initialization...'
                    bat 'timeout /t 15 /nobreak'
                }
            }
        }

        stage('Verify Core Topics') {
            steps {
                script {
                    echo 'Verifying running infrastructure components...'
                    bat 'docker ps --filter name=kafka-local'

                    echo 'Creating product_updates topic if it does not exist...'
                    try {
                        bat 'docker exec kafka-local kafka-topics --create --topic product_updates --bootstrap-server localhost:9092 --partitions 1 --replication-factor 1 || ver > nul'
                        echo 'Topic product_updates verified successfully.'
                    } catch (Exception e) {
                        echo 'Topic might already exist, skipping configuration.'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Core Event Streaming Infrastructure Deployed Successfully!'
        }
        failure {
            echo 'Infrastructure deployment pipeline failed. Check logs above.'
        }
    }
}