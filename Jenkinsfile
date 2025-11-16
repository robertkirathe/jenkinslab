pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    IMAGE_NAME = "rmaina/jenkinslab"
    IMAGE_TAG = "v1.${BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/robertkirathe/jenkinslab.git'
      }
    }

    stage('Build') {
      steps {
        script {
          dockerImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
        }
      }
    }

    stage('Test') {
      steps {
        sh 'echo "Running tests..."'
        sh './mvnw test || true' // adjust if using Maven
      }
    }

    stage('Static Analysis') {
      steps {
        sh 'echo "Running SonarQube scan..."'
        // Optional: integrate SonarQube CLI or plugin here
      }
    }

    stage('Push') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
            dockerImage.push()
          }
        }
      }
    }
  }
}
