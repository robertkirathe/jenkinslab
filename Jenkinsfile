pipeline {
    agent {
        kubernetes {
            inheritFrom 'default'
        }
    }

    environment {
        DOCKER_IMAGE = "rmaina/jenkinslab:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                // run in the jnlp container
                container('jnlp') {
                    git url: 'https://github.com/robertkirathe/jenkinslab.git', branch: 'main'
                }
            }
        }

        stage('Test') {
            steps {
                container('jnlp') {
                    sh '''
                        if ! command -v mvn >/dev/null; then
                            echo "Installing Maven..."
                            apt-get update && apt-get install -y maven
                        fi
                        ./mvnw -B test
                    '''
                }
            }
        }

        stage('Static Analysis - SonarQube') {
            steps {
                container('jnlp') {
                    sh '''
                        echo "Running SonarQube scan..."
                        # Replace with your SonarQube scan command if needed
                        ./mvnw sonar:sonar || true
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('jnlp') {
                    sh '''
                        if ! command -v docker >/dev/null; then
                            echo "Installing Docker CLI..."
                            apt-get update && apt-get install -y docker.io
                        fi
                        docker build -t ${DOCKER_IMAGE} .
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('jnlp') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh '''
                            echo "$PASS" | docker login -u "$USER" --password-stdin
                            docker push ${DOCKER_IMAGE}
                        '''
                    }
                }
            }
        }

    }

    post {
        success {
            echo "Pipeline completed successfully. Docker image pushed to Docker Hub: ${DOCKER_IMAGE}"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }

}
