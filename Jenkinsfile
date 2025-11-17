pipeline {
    agent {
        kubernetes {
            label 'jenkinslab-pipeline'
            defaultContainer 'jnlp'
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kankio
    image: gcr.io/kaniko-project/executor:latest
    command:
    - /busybox/sh
    args:
    - -c
    - "sleep 99d"
    volumeMounts:
    - name: kaniko-secret
      mountPath: /kaniko/.docker/
  volumes:
  - name: kaniko-secret
    emptyDir: {}
"""
        }
    }

    environment {
        DOCKER_IMAGE = "rmaina/jenkinslab:latest"
        DOCKER_CREDENTIALS_ID = "dockerhub-credentials" // Make sure this matches your Jenkins secret
        SONARQUBE_SERVER = "SonarQube" // Replace with your Jenkins SonarQube server ID
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/robertkirathe/jenkinslab.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                container('maven') {
                    sh './mvnw clean package -B'
                }
            }
        }

        stage('Test') {
            steps {
                container('maven') {
                    sh './mvnw test -B'
                }
            }
        }

        stage('Static Analysis') {
            steps {
                script {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        container('maven') {
                            sh './mvnw sonar:sonar -B'
                        }
                    }
                }
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                container('kankio') {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh """
                        echo '{ "auths": { "https://index.docker.io/v1/": { "auth": "'\$(echo -n \$USERNAME:\$PASSWORD | base64)'" } } }' > /kaniko/.docker/config.json
                        /kaniko/executor \\
                            --dockerfile=Dockerfile \\
                            --context=dir://workspace \\
                            --destination=${DOCKER_IMAGE} \\
                            --skip-tls-verify
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
