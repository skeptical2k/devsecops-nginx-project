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
                sh 'docker rm -f nginx-app 2>/dev/null || true'
                sh 'docker compose up -d --build nginx-app'
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 10'
                sh 'docker ps'
                sh 'docker exec nginx-app wget -qO- http://localhost'

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
