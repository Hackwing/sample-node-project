pipeline {
    agent {
        label 'docker-agent'
    }

    tools {
        nodejs 'NodeJS' // ensure NodeJS is configured in Jenkins global tools
    }

    environment {
        SONARQUBE_ENV = 'SonarQube' // this must match the configured SonarQube installation name in Jenkins
        IMAGE_NAME = 'sample-node-project'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${env.SONARQUBE_ENV}") {
                    sh 'npx sonar-scanner'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh "trivy image ${IMAGE_NAME}:${env.BUILD_NUMBER} || true"
            }
        }

        stage('Deploy Locally') {
            steps {
                sh "docker run -d -p 3000:3000 ${IMAGE_NAME}:${env.BUILD_NUMBER}"
            }
        }
    }

    post {
        failure {
            echo 'Build failed!'
        }
        success {
            echo 'Build succeeded!'
        }
    }
}
