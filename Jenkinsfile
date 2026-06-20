pipeline {

    agent any

    environment {
        IMAGE_NAME = "scl"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ./app
                '''
            }
        }

        stage('Deploy Green') {
            steps {
                sh '''
                docker rm -f scl-green || true

                docker run -d \
                  --name scl-green \
                  --network scl-net \
                  ${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 10

                docker exec scl-green \
                  curl -f http://localhost
                '''
            }
        }

        stage('Switch Traffic') {
            steps {
                sh '''
                sed -i 's/scl-blue/scl-green/g' nginx/nginx.conf

                docker exec scl-proxy nginx -t
                docker exec scl-proxy nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed"
        }
    }
}
