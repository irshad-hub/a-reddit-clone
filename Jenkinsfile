pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "reddit-clone-pipeline"
        RELEASE = "1.0.0"
        DOCKER_REGISTRY = 'https://hub.docker.com/'  // Docker Hub registry
        DOCKER_USER = 'irshadahmed'
        DOCKER_PASS = 'dckr_pat_fOkem3FMwoeJ_y6z6kyjZsc8vWc'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/irshad-hub/a-reddit-clone.git'
            }
        }
        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit-clone-CI \
                    -Dsonar.projectKey=Reddit-clone-CI'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
             }
         }
        stage('build and push docker image') {
            steps {
                script {
                    def dockerImage

                    // Log in to Docker Hub
                    sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"

                    // Build Docker image
                    dockerImage = docker.build "${IMAGE_NAME}:${IMAGE_TAG}"

                    // Push Docker image
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ('latest')"
                    
                   
                }
             }
         }
         stage('TRIVY image SCAN') {
            steps {
                script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image irshadahmed/reddit-clone-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
                }
             }
         }
        stage ('Cleanup Artifacts') {
             steps {
                 script {
                      sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                      sh "docker rmi ${IMAGE_NAME}:latest"
                 }
             }
         }
    
   }
}
