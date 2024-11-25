pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "back-app"
        RABBITMQ_IMAGE = "rabbitmq:3-management"
        INVENTORY = "PinSenderBackend-main/inventory.ini"
        PLAYBOOK = "PinSenderBackend-main/deploy.yml"
        TEST_HOST = "app"
        MAVEN_OPTS = '-Dmaven.repo.local=/var/lib/jenkins/.m2/repository'
    }

    tools {
        maven 'Maven-3.8.5'
    }

    stages {
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/ahmedenzo/pipeback.git'
            }
        }

        stage('Vérifier les fichiers nécessaires') {
            steps {
                script {
                    echo "Vérification des fichiers nécessaires : ${INVENTORY} et ${PLAYBOOK}..."

                    if (!fileExists("${INVENTORY}")) {
                        error "Le fichier ${INVENTORY} est introuvable. Arrêt du pipeline."
                    } else {
                        echo "Le fichier ${INVENTORY} est présent."
                    }

                    if (!fileExists("${PLAYBOOK}")) {
                        error "Le fichier ${PLAYBOOK} est introuvable. Arrêt du pipeline."
                    } else {
                        echo "Le fichier ${PLAYBOOK} est présent."
                    }
                }
            }
        }

        stage('Test Ansible Connection') {
            steps {
                script {
                    def connectionStatus = sh(
                        script: '''
                            echo "Testing Ansible connection..."
                            ansible all -m ping -i ${INVENTORY}
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
                    sh '''
                        docker build -t ${BACKEND_IMAGE}:latest .
                    '''
                }
            }
        }

        stage('Pull RabbitMQ Docker Image') {
            steps {
                sh '''
                    docker pull ${RABBITMQ_IMAGE}
                '''
            }
        }

        stage('Export Docker Images') {
            steps {
                dir('PinSenderBackend-main') {
                    sh '''
                        docker save -o backend-image.tar ${BACKEND_IMAGE}:latest
                        docker save -o rabbitmq-image.tar ${RABBITMQ_IMAGE}
                    '''
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                dir('PinSenderBackend-main') {
                    sh '''
                        ansible-playbook -i inventory.ini deploy.yml
                    '''
                }
            }
        }

        stage('Clean Local Docker Images') {
            steps {
                sh '''
                    docker rmi -f ${BACKEND_IMAGE}:latest || true
                    docker rmi -f ${RABBITMQ_IMAGE} || true
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded! 🎉'
        }
        failure {
            echo 'Pipeline failed. ❌'
        }
    }
}
