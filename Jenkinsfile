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
                container('jnlp') {
                    git url: 'https://github.com/robertkirathe/jenkinslab.git', branch: 'main'
                }
            }
        }

        stage('Test') {
            steps {
                container('maven') {
                    sh "./mvnw -B test"
                }
            }
        }

        stage('Static Analysis - SonarQube') {
            environment {
                SONAR_HOST_URL = credentials('sonar-host')
                SONAR_TOKEN    = credentials('sonar-token')
            }
            steps {
                container('maven') {
                    sh """
                    ./mvnw sonar:sonar \
                      -Dsonar.host.url=$SONAR_HOST_URL \
                      -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                container('docker') {
                    sh "docker build -t $DOCKER_IMAGE ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )]) {
                        sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                        docker push $DOCKER_IMAGE
                        """
                    }
                }
            }
        }

    }

    post {
        success {
            echo "Pipeline completed successfully. Docker image pushed to Docker Hub as rmaina/jenkinslab:latest"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }

}
