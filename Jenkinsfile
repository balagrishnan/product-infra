pipeline {
    agent any

    stages {
        stage('Checkout Infrastructure Repo') {
            steps {
                cleanWs()
                checkout scm
            }
        }

        stage('Pre-Cleanup Conflicts') {
            steps {
                script {
                    echo 'Force removing any pre-existing conflicting containers...'
                    // This stops and removes any loose containers using the conflicting names
                    try { bat 'docker rm -f kafka-broker || ver > nul' } catch(Exception e) {}
                    try { bat 'docker rm -f zookeeper-local || ver > nul' } catch(Exception e) {}
                }
            }
        }

        stage('Deploy Infra via Docker Compose') {
            steps {
                script {
                    echo 'Stopping and cleaning this projects previous compose stack...'
                    bat 'docker-compose down -v --remove-orphans || ver > nul'

                    echo 'Launching Kafka & Zookeeper Core Services...'
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
                    bat 'docker ps --filter name=kafka-broker'
                }
            }
        }
    }
}