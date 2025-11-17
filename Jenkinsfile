pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: slave
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:3345.v03dee9b_f88fc-1
    args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
    tty: true
  - name: kankio
    image: gcr.io/kaniko-project/executor:latest
    command: ["/busybox/sh"]
    args: ["sleep", "infinity"]
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker/
  volumes:
    - name: kaniko-secret
      secret:
        secretName: dockerhub-secret
"""
        }
    }

    environment {
        DOCKER_IMAGE = "rmaina/jenkinslab:latest"
        DOCKER_CREDENTIALS_ID = "dockerhub"   // Your Jenkins DockerHub credentials ID
        SONARQUBE_SERVER = "SonarQube"        // Your SonarQube server in Jenkins
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/robertkirathe/jenkinslab.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                container('kankio') {
                    sh """
                    /kaniko/executor \
                        --dockerfile=Dockerfile \
                        --context=dir://workspace \
                        --destination=${DOCKER_IMAGE} \
                        --skip-tls-verify
                    """
                }
            }
        }

        stage('Test') {
            steps {
                container('jnlp') {
                    sh './mvnw test'
                }
            }
        }

        stage('Static Analysis') {
            steps {
                container('jnlp') {
                    withSonarQubeEnv("${SONARQUBE_SERVER}") {
                        sh './mvnw sonar:sonar'
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                container('kankio') {
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh """
                        echo "{\"auths\":{\"https://index.docker.io/v1/\":{\"auth\":\"$(echo -n $USERNAME:$PASSWORD | base64)\"}}}" > /kaniko/.docker/config.json
                        /kaniko/executor \
                            --dockerfile=Dockerfile \
                            --context=dir://workspace \
                            --destination=${DOCKER_IMAGE} \
                            --skip-tls-verify
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs!'
        }
    }
}
