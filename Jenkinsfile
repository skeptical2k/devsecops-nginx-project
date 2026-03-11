pipeline {
    agent any

    environment {
        IMAGE_NAME = "devsecops-nginx"
        IMAGE_TAG = "latest"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Validate Files') {
            steps {
                sh 'echo "Checking project files..."'
                sh 'ls -R'
                sh 'test -f Dockerfile'
                sh 'test -f docker-compose.yml'
                sh 'test -f nginx/default.conf'
                sh 'test -f html/index.html'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $FULL_IMAGE .'
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker compose up -d --build nginx-app'
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 5'
                sh 'curl -f http://localhost:8088'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
