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
                sh '''
                    python3 -m pip install -r requirements.txt
                    python3 -m unittest discover -s tests
                '''
            }
        }

        stage('Deploy') {
            steps {
                echo "Fazendo deploy da aplicação (container local)..."
                script {
                    sh "docker run -d --rm --name meuapp_container -p 5000:5000 ${IMAGE_NAME}:${DOCKER_TAG}"
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
