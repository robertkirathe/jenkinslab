pipeline {
    agent {
        kubernetes {
            label 'jenkinslab-pipeline'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: slave
spec:
  containers:
  - name: kankio
    image: gcr.io/kaniko-project/executor:latest
    # Kaniko executor entrypoint is already /kaniko/executor, no need for /busybox/sh
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker/
  - name: maven
    image: maven:3.9.5-eclipse-temurin-17
    command:
    - cat
    tty: true
  volumes:
  - name: kaniko-secret
    secret:
      secretName: docker-config
"""
        }
    }

    environment {
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_REPO     = 'your-dockerhub-username'
        IMAGE_NAME      = 'jenkinslab'
        IMAGE_TAG       = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                container('jnlp') {
                    checkout scm
                }
            }
        }

        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package -B'
                }
            }
        }

        stage('Test') {
            steps {
                container('maven') {
                    sh 'mvn test -B'
                }
            }
        }

        stage('Static Analysis') {
            steps {
                container('maven') {
                    // Adjust SonarQube environment and token
                    withSonarQubeEnv('SonarQube') {
                        sh 'mvn sonar:sonar -B'
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                container('kankio') {
                    sh """
                    /kaniko/executor \\
                        --dockerfile=Dockerfile \\
                        --context=\$WORKSPACE \\
                        --destination=${DOCKER_REPO}/${IMAGE_NAME}:${IMAGE_TAG} \\
                        --insecure
                    """
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished. Cleaning up..."
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed. Check logs."
        }
    }
}
