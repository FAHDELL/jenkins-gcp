pipeline {
    agent any

    tools {
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
                checkout scmGit(
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        credentialsId: 'jenkins-gcp',
                        url: 'https://github.com/FAHDELL/jenkins-gcp.git'
                    ]]
                )
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building Java App with Maven'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('JUnit Tests') {
            steps {
                echo 'Running Unit Tests'
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube Analysis'
                withCredentials([string(credentialsId: 'sonartoken', variable: 'sonarToken')]) {
                    withSonarQubeEnv('sonar') {
                        sh '''
                        ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                          -Dsonar.projectKey=jenkinsgcp \
                          -Dsonar.sources=. \
                          -Dsonar.java.binaries=target/classes \
                          -Dsonar.token=$sonarToken
                        '''
                    }
                }
            }
        }

        stage('Trivy FS Scan') {
            steps {
                echo 'Scanning File System with Trivy'
                sh '''
                trivy fs \
                  --format table \
                  --output FSScanReport.txt \
                  .
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image'
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo 'Scanning Docker Image with Trivy'
                sh '''
                trivy image \
                  --timeout 30m \
                  --severity HIGH,CRITICAL \
                  --cache-dir ${WORKSPACE}/.trivy-cache \
                  --format table \
                  --output trivyImageScanReport.txt \
                  ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Azure Steps (Placeholder)') {
            steps {
                echo 'Authenticate + Push to ACR + Deploy to ACI'
            }
        }
    }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
    }

    post {
        always {
            archiveArtifacts artifacts: '''
                FSScanReport.txt,
                trivyImageScanReport.txt
            ''', fingerprint: true
        }

        success {
            echo 'Pipeline SUCCESS ✅'
        }

        failure {
            echo 'Pipeline FAILED ❌ - check reports'
        }
    }
}
