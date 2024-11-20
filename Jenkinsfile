pipeline {
    agent any

    tools {
        maven 'Maven-3.8.5' // Maven configuré dans Jenkins
    }

    environment {
        MAVEN_OPTS = '-Dmaven.repo.local=/var/lib/jenkins/.m2/repository' // Utilisation du cache global Maven
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Cloner le repository Git
                git branch: 'master', url: 'https://github.com/ahmedenzo/pipeback.git'
            }
        }

        stage('Build with Maven') {
            steps {
                dir('PinSenderBackend-main') {
                    // Construire le projet avec Maven (sans tests)
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                // Construire l'image Docker en ciblant le Dockerfile dans 'PinSenderBackend-main'
                sh 'docker build -t backend-image:latest PinSenderBackend-main'
            }
        }

        stage('Pull RabbitMQ Image') {
            steps {
                // Tirer l'image Docker officielle de RabbitMQ
                sh 'docker pull rabbitmq:3-management'
            }
        }
    }

    post {
        always {
            // Nettoyage pour libérer de l'espace disque
            sh 'docker system prune -f || true'
        }
    }
}
