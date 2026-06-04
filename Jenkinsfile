pipeline {
    agent any
    tools{
        jdk 'java21.0.11'
        maven 'maven399'
    }
    environment {
        SONAR_SCANNER_HOME = tool 'sonar8'
        IMAGE_NAME = "java-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    stages {

        stage('Initialize Pipeline') {
            steps {
                echo 'Initializing Pipeline ...'
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Checkout GitHub Codes') {
            steps {
                echo 'Checking out GitHub Codes'
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'jenkins-gcp', url: 'https://github.com/FAHDELL/jenkins-gcp.git']])
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building Java App with Maven'
                sh 'mvn clean package'
            }
        }

        stage('JUnit Test of Java App') {
            steps {
                echo 'JUnit Testing'
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running Static Code Analysis with SonarQube'
                withCredentials([string(credentialsId: 'sonartoken', variable: 'sonarToken')]) {
                    withSonarQubeEnv('sonar') {
                        sh '''
                           ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=jenkinsgcp \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=http://172.18.0.3:9000 \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.token=$sonarToken
                        '''
                    }
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                echo 'Scanning File System with Trivy FS ...'
                    sh '''
                        trivy fs \
                          --format table \
                          --output FSScanReport.html \
                          .
                        '''
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                echo 'Building the Java App Docker Image'
                script {
                    sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo 'Scanning Docker Image with Trivy'
                sh '''
                    trivy image \
                      --timeout 30m \
                      --severity HIGH,CRITICAL \
                      --cache-dir ${WORKSPACE}/.trivy-cache \
                      --no-progress \
                      --format table \
                      -o trivyImageScanReport.html \
                      ${IMAGE_NAME}:${IMAGE_TAG}
                    '''
            }
        }

        stage('Authenticate with Azure') {
            steps {
                echo 'Authenticating with Azure'
            }
        }

        stage('Tag & Push Image to Azure Container Registry (ACR)') {
            steps {
                echo 'Tagging and Pushing Image to ACR'
                echo "Pushing Docker image to ACR..."
            }
        }

        stage('Deploy to Azure Container Instance (ACI) & Get App URL') {
            steps {
                echo 'Deploying Image to ACI'
            }
        }
    }
}
