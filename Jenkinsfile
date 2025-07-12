pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'SonarQube Server' // Must match name in Jenkins config
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/acemilyalcin/sample-node-project.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('My SonarQube Server') {
                    sh 'sonar-scanner'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t sample-node-project .'
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh 'trivy image sample-node-project || true'
            }
        }

        stage('Deploy Locally') {
            steps {
                sh 'docker run -d -p 3000:3000 --name sample-node sample-node-project'
            }
        }
    }
}
