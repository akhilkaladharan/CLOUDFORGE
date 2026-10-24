pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out CloudForge source code'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t cloudforge-cloudforge:latest .'
            }
        }

        stage('Deploy CloudForge') {
            steps {
                sh 'docker compose up -d --build'
            }
        }
    }

    post {
        success {
            echo 'CloudForge deployed successfully!'
        }

        failure {
            echo 'CloudForge pipeline failed. Check the console output.'
        }
    }
}
