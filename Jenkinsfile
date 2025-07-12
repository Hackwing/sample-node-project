pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'Sonarqube1' // Must match Jenkins' SonarQube config name
    }

    options {
        // Retry entire pipeline if transient error occurs
        retry(2)
        // Keep only last 5 builds
        buildDiscarder(logRotator(numToKeepStr: '5'))
        // Fail fast on first stage failure
        disableConcurrentBuilds()
        timeout(time: 15, unit: 'MINUTES')
    }

    stages {

        stage('Checkout') {
            agent {
                label any
            }
            options {
                retry(2) // Prevents resume error after Jenkins restart
            }
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
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'npm run sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = "sample-node-project:${env.BUILD_NUMBER}"
                    sh "docker build -t ${imageName} ."
                }
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh '''
                docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image sample-node-project:${BUILD_NUMBER}
                '''
            }
        }

        stage('Deploy Locally') {
            steps {
                sh 'docker run -d -p 3000:3000 sample-node-project:${BUILD_NUMBER}'
            }
        }
    }

    post {
        always {
            echo "Cleaning up resources..."
            sh 'docker system prune -f || true'
        }
        success {
            echo '✅ Build and deployment succeeded!'
        }
        failure {
            echo '❌ Build failed!'
        }
        unstable {
            echo '⚠️ Build is unstable.'
        }
    }
}
