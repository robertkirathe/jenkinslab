environment {
    DOCKER_USERNAME = "rmaina" 
    IMAGE_NAME = "docker.io/${env.DOCKER_USERNAME}/jenkinslab-petclinic"
    IMAGE_TAG = "v${env.BUILD_NUMBER}"
}

stages {
    stage('Build') {
        agent {
            kubernetes {
                yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  serviceAccountName: jenkins
                  containers:
                  - name: maven
                    image: maven:3.9-eclipse-temurin-17
                    imagePullPolicy: IfNotPresent
                    command:
                    - sleep
                    args:
                    - 999999
                    # --- THIS IS THE FIX for 'Pod [Pending]' ---
                    resources:
                      requests:
                        memory: "256Mi"
                        cpu: "250m"
                      limits:
                        memory: "512Mi"
                        cpu: "500m"
                """
            }
        }
        steps {
            container(name: 'maven') {
                echo "--- Making Maven Wrapper executable ---"
                sh 'chmod +x mvnw'
                
                echo "--- Building the JAR file with Maven Wrapper ---"
                sh './mvnw clean package -DskipTests'
                
                echo "--- Stashing build artifacts for next stage ---"
                stash includes: 'target/*.jar', name: 'jar-file'
                stash includes: 'Dockerfile', name: 'dockerfile'
                // Stash the whole project for the sonar stage
                stash includes: '**/*', name: 'project-source'
            }
        }
    }

    // --- THIS STAGE WAS MISSING ---
    stage('SonarQube Analysis') {
        agent {
            kubernetes {
                yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  serviceAccountName: jenkins
                  containers:
                  - name: maven
                    image: maven:3.9-eclipse-temurin-17
                    imagePullPolicy: IfNotPresent
                    command:
                    - sleep
                    args:
                    - 999999
                    # --- THIS IS THE FIX for 'Pod [Pending]' ---
                    resources:
                      requests:
                        memory: "256Mi"
                        cpu: "250m"
                      limits:
                        memory: "512Mi"
                        cpu: "500m"
                """
            }
        }
        steps {
            container(name: 'maven') {
                unstash 'project-source'
                // This 'SonarQube' name MUST match the name in your Jenkins config
                withSonarQubeEnv('SonarQube') {
                    echo "--- Running SonarQube Analysis ---"
                    sh './mvnw sonar:sonar'
                }
            }
        }
    }

    stage('Build & Push Docker Image') {
        agent { 
            kubernetes {
                yaml """
                apiVersion: v1
                kind: Pod
                spec:
                  serviceAccountName: jenkins
                  containers:
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:latest
                    imagePullPolicy: Always
                    command:
                    - /busybox/cat
                    tty: true
                    volumeMounts:
                      - name: docker-config
                        mountPath: /kaniko/.docker
                    # --- THIS IS THE FIX for 'Pod [Pending]' ---
                    resources:
                      requests:
                        memory: "256Mi"
                        cpu: "250m"
                      limits:
                        memory: "512Mi"
                        cpu: "500m"
                  volumes:
                    - name: docker-config
                      secret:
                        secretName: docker-config-secret
                """
            }
        }
        steps {
            container(name: 'kaniko', shell: '/busybox/sh') {
                echo "--- Unstashing artifacts ---"
                sh 'mkdir -p target'
                unstash name: 'jar-file'
                unstash name: 'dockerfile'

                echo "--- Building and Pushing Image ---"
                sh """
                /kaniko/executor \
                  --context=. \
                  --dockerfile=Dockerfile \
                  --destination=${env.IMAGE_NAME}:${env.IMAGE_TAG} \
                  --build-arg JAR_FILE=target/*.jar
                """
            }
        }
    }
}

post {
    always {
        echo "Cleaning up workspace..."
        cleanWs()
    }
}
