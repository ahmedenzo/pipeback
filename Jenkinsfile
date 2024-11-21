pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "back-app" // Nom de l'image backend
        RABBITMQ_IMAGE = "rabbitmq:3-management" // Nom de l'image RabbitMQ
        INVENTORY = "inventory.ini" // Relative to 'PinSenderBackend-main'
        PLAYBOOK = "deploy.yml"    // Relative to 'PinSenderBackend-main'
        TEST_HOST = "10.0.2.10" // Adresse de la VM backend cible
        MAVEN_OPTS = '-Dmaven.repo.local=/var/lib/jenkins/.m2/repository' // Cache Maven
    }

    tools {
        maven 'Maven-3.8.5' // Maven configuré dans Jenkins
    }

    stages {
        stage('Test Ansible Connection') {
            steps {
                script {
                    def connectionStatus = sh(
                        script: '''
                            echo "Testing Ansible connection..."
                            sudo ansible -i PinSenderBackend-main/${INVENTORY} ${TEST_HOST} -m ping
                        ''',
                        returnStatus: true
                    )
                    if (connectionStatus != 0) {
                        error "Ansible connection test failed! Skipping pipeline."
                    } else {
                        echo "Ansible connection successful."
                    }
                }
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/ahmedenzo/pipeback.git'
            }
        }

        stage('Build with Maven') {
            steps {
                dir('PinSenderBackend-main') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                dir('PinSenderBackend-main') {
                    sh "docker build -t ${BACKEND_IMAGE}:latest ."
                }
            }
        }

        stage('Pull RabbitMQ Docker Image') {
            steps {
                sh "docker pull ${RABBITMQ_IMAGE}"
            }
        }

        stage('Export Docker Images') {
            steps {
                dir('PinSenderBackend-main') {
                    sh '''
                        echo "Exporting backend Docker image..."
                        docker save -o backend-image.tar ${BACKEND_IMAGE}:latest
                        echo "Exporting RabbitMQ Docker image..."
                        docker save -o rabbitmq-image.tar ${RABBITMQ_IMAGE}
                    '''
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                dir('PinSenderBackend-main') {
                    sh '''
                        echo "Executing Ansible playbook to deploy images..."
                        sudo ansible-playbook -i ${INVENTORY} ${PLAYBOOK}
                    '''
                }
            }
        }

        stage('Clean Local Docker Images') {
            steps {
                sh '''
                    echo "Cleaning up local Docker images..."
                    docker rmi -f ${BACKEND_IMAGE}:latest || true
                    docker rmi -f ${RABBITMQ_IMAGE} || true
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! Images exported, deployed, and cleaned up successfully. 🎉'
        }
        failure {
            echo 'Pipeline failed! ❌'
        }
    }
}
