pipeline {
    agent any

    environment {
        IMAGE_NAME = "meuapp"
        DOCKER_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('SAST - Semgrep') {
            steps {
                echo "Analisando código com Semgrep..."
                sh '''
                    curl -sL https://semgrep.dev/install.sh | bash
                    ./semgrep/semgrep scan --config auto .
                '''
            }
        }

        stage('Build da Imagem') {
            steps {
                echo "Criando imagem Docker..."
                script {
                    docker.build("${IMAGE_NAME}:${DOCKER_TAG}")
                }
            }
        }

        stage('SCA - Trivy') {
            steps {
                echo "Escaneando imagem com Trivy..."
                sh '''
                    if ! command -v trivy >/dev/null 2>&1; then
                        echo "Instalando Trivy..."
                        wget https://github.com/aquasecurity/trivy/releases/latest/download/trivy_0.50.1_Linux-64bit.deb
                        sudo dpkg -i trivy_0.50.1_Linux-64bit.deb
                    fi
                    trivy image ${IMAGE_NAME}:${DOCKER_TAG}
                '''
            }
        }

        stage('Testes Automatizados') {
            steps {
                echo "Executando unittest no container..."
                script {
                    docker.image("${IMAGE_NAME}:${DOCKER_TAG}").inside {
                        sh '''
                            python3 -m unittest discover -s tests
                        '''
                    }
                }
            }
        }

        stage('Deploy Local') {
            steps {
                echo "Subindo container local para testes dinâmicos..."
                script {
                    def containerName = "meuapp_container_${BUILD_ID}"
                    sh "docker run -d --rm --name ${containerName} -p 5000:5000 ${IMAGE_NAME}:${DOCKER_TAG}"
                    sleep(time: 10, unit: "SECONDS") // aguarda app subir
                }
            }
        }

        stage('DAST - OWASP ZAP') {
            steps {
                echo "Escaneando app com OWASP ZAP..."
                sh '''
                    docker run --network host -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:5000 || true
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline finalizada!"
        }
    }
}
