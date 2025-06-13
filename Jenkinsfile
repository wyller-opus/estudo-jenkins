pipeline {
    agent any

    environment {
        IMAGE_NAME = "meuapp"
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Realizando checkout do código..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Fazendo build da imagem Docker..."
                script {
                    docker.build("${IMAGE_NAME}:${DOCKER_TAG}")
                }
            }
        }

        stage('Test') {
            steps {
                echo "Executando testes..."
                script {
                    docker.image("${IMAGE_NAME}:${DOCKER_TAG}").inside {
                        sh '''
                            python3 -m unittest discover -s tests
                        '''
                    }
                }
            }
        }


        stage('Deploy') {
            steps {
                echo "Fazendo deploy da aplicação (container local)..."
                script {
                    def containerName = "meuapp_container_${BUILD_ID}"
                    sh "docker run -d --rm --name ${containerName} -p 5000:5000 ${IMAGE_NAME}:${DOCKER_TAG}"
                    echo "Container iniciado: ${containerName}"
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finalizado"
        }
    }
}
