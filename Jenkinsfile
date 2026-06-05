pipeline {
    agent any

    tools {
        jdk 'java 21.0.11'
        maven 'Maven 3.9.9'
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
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('JUnit Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonarToken', variable: 'sonarToken')]) {
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
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Image Scan') {
            steps {
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

        stage('Deploy to Kubernetes') {
            steps {
                // 1. Téléchargement de kubectl
                sh 'curl -LO "https://dl.k8s.io/release/\$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                sh 'chmod +x ./kubectl'
                
                // 2. Utilisation directe du fichier secret avec le nouvel ID
                withCredentials([file(credentialsId: 'minikube-flat-config', variable: 'KUBECONFIG_FILE')]) {
                    sh './kubectl --kubeconfig=$KUBECONFIG_FILE apply -f k8s/deployment.yaml'
                    sh './kubectl --kubeconfig=$KUBECONFIG_FILE apply -f k8s/service.yaml'
                    sh './kubectl --kubeconfig=$KUBECONFIG_FILE rollout status deployment/java-app'
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                withCredentials([file(credentialsId: 'minikube-flat-config', variable: 'KUBECONFIG_FILE')]) {
                    sh './kubectl --kubeconfig=$KUBECONFIG_FILE get pods'
                    sh './kubectl --kubeconfig=$KUBECONFIG_FILE get svc'
                }
            }
        }

        stage('Azure Steps (Placeholder)') {
            steps {
                echo 'Authenticate + Push to ACR + Deploy to ACI'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '''
                FSScanReport.txt,
                trivyImageScanReport.txt
            ''', fingerprint: true, allowEmptyArchive: true
        }

        success {
            echo 'Pipeline SUCCESS ✅'
        }

        failure {
            echo 'Pipeline FAILED ❌ - check reports'
        }
    }
}
