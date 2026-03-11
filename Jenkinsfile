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
                sh 'test -f prometheus/prometheus.yml'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $FULL_IMAGE .'
            }
        }

        stage('Security Scan with Trivy') {
            steps {
                sh '''
                docker run --rm \
                  -v /var/run/docker.sock:/var/run/docker.sock \
                  aquasec/trivy:latest image $FULL_IMAGE
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh 'docker compose up -d --build nginx-app nginx-exporter prometheus grafana'

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
