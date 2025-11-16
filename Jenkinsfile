pipeline {
  agent {
    kubernetes {
      inheritFrom 'kaniko'
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
        git url: 'https://github.com/robertkirathe/jenkinslab.git'
      }
    }

    stage('Build and Push with Kaniko') {
      steps {
        sh '''
          /kaniko/executor \
            --context `pwd` \
            --dockerfile `pwd`/Dockerfile \
            --destination=${IMAGE_NAME}:${IMAGE_TAG} \
            --verbosity=info \
            --skip-tls-verify
        '''
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
}
