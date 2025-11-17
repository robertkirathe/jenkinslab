pipeline {
    agent {
        kubernetes {
            label 'jenkinslab-agent'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: jnlp
      image: jenkins/inbound-agent:3345.v03dee9b_f88fc-1
      args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
      tty: true
      command:
        - cat
"""
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
            echo "Pipeline completed successfully. Docker image pushed: ${DOCKER_IMAGE}"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }

}
