pipeline {
    agent {
        kubernetes {
            inheritFrom 'jenkinslab-agent'  // Use the pod template label you configured
            defaultContainer 'kaniko'
        }
    }

    environment {
        IMAGE_NAME = "rmaina/jenkinslab"
        IMAGE_TAG  = "v1.${BUILD_NUMBER}"
        DOCKER_CONFIG = '/kaniko/.docker/'  // Kaniko Docker credentials path
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/robertkirathe/jenkinslab.git', branch: 'main'
            }
        }

        stage('Build & Push with Kaniko') {
            steps {
                container('kaniko') {
                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds', 
                        usernameVariable: 'DOCKER_USERNAME', 
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh '''
                        mkdir -p /kaniko/.docker
                        echo "{\"auths\":{\"https://index.docker.io/v1/\":{\"username\":\"$DOCKER_USERNAME\",\"password\":\"$DOCKER_PASSWORD\"}}}" > /kaniko/.docker/config.json

                        /kaniko/executor \
                            --context `pwd` \
                            --dockerfile `pwd`/Dockerfile \
                            --destination=${IMAGE_NAME}:${IMAGE_TAG} \
                            --verbosity=info \
                            --skip-tls-verify
                        '''
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh './mvnw test || true'
            }
        }

        stage('Static Analysis') {
            steps {
                sh 'echo "SonarQube scan placeholder..."'
            }
        }
    }

    post {
        success {
            echo "Pipeline finished successfully. Image pushed: ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
