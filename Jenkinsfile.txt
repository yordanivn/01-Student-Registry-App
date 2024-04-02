pipeline {
    agent any

    environment {
        NODE_VERSION = '21.7.1'
        SERVICE_ID = 'render-service-id'
        RENDER_API_KEY = 'render-api-key'
    }

    tools {
        nodejs "${NODE_VERSION}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                bat 'npm install'
                bat 'npm install -g wait-on kill-port'
            }
        }

        stage('Run tests') {
            steps {
                bat 'start /b npm run start'
                bat 'wait-on http://localhost:8081'
                bat 'npm test'
                bat 'kill-port 8081'
            }
        }
    }

    post {
        always {
            echo 'CI Pipeline completed.'
        }
        failure {
            bat 'kill-port 8081'
        }
    }
}