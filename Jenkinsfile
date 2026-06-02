pipeline {
    agent any
    tools{
        jdk 'java21.0.11'
        maven 'maven399'
    }

    stages {

        stage('Initialize Pipeline') {
            steps {
                echo 'Initializing Pipeline ...'
                sh 'java -version'
                sh 'maven -version'
            }
        }

        stage('Checkout GitHub Codes') {
            steps {
                echo 'Checking out GitHub Codes'
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building Java App with Maven'
            }
        }

        stage('JUnit Test of Java App') {
            steps {
                echo 'JUnit Testing'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running Static Code Analysis with SonarQube'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                echo 'Scanning File System with Trivy FS ...'
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                echo 'Building the Java App Docker Image'
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo 'Scanning Docker Image with Trivy'
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
