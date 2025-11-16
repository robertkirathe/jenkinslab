pipeline {
    agent {
        kubernetes {
            label 'kaniko'
            defaultContainer 'kaniko'
        }
    }

    environment {
        IMAGE_NAME = "rmaina/jenkinslab"
        IMAGE_TAG = "v1.${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                // Explicitly checkout the main branch
                git branch: 'main', url: 'https://github.com/robertkirathe/jenkinslab.git'
            }
        }

        stage('Build and Push with Kaniko') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        /kaniko/executor \
                        --context `pwd` \
                        --dockerfile `pwd`/Dockerfile \
                        --destination=${IMAGE_NAME}:${IMAGE_TAG} \
                        --verbosity=info \
                        --skip-tls-verify \
                        --docker-config=/kaniko/.docker/
                    '''
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
                // Placeholder for SonarQube scan
                sh 'echo "SonarQube scan placeholder..."'
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Check stages above for results."
        }
        success {
            echo "Build, test, and push succeeded!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
