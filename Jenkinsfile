pipeline {
    agent any

    environment {
        // Update this to match your Jenkins credentials ID for Docker Hub
        DOCKERHUB_CREDENTIALS = 'dockerhub'
        DOCKER_IMAGE = 'django-todo'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Pratik543/django-notes-app-ci-cd'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker Image..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                        sh "docker build -t ${DOCKERHUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    }
                }
            }
        }

        stage('Run Docker Compose'){
            steps{
                script{
                    echo "Running Docker Compose..."
                    sh "docker compose up -d"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    echo "Pushing Docker Image to Docker Hub..."
                    withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                        sh "echo \$DOCKERHUB_PASS | docker login -u \$DOCKERHUB_USER --password-stdin"
                        sh "docker push ${DOCKERHUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG}"
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                echo "Cleaning up local Docker images..."
                withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", passwordVariable: 'DOCKERHUB_PASS', usernameVariable: 'DOCKERHUB_USER')]) {
                    sh "docker rmi ${DOCKERHUB_USER}/${DOCKER_IMAGE}:${DOCKER_TAG} || true"
                }
            }
        }
        success {
            echo "CI/CD Pipeline execution successful!"
        }
        failure {
            echo "CI/CD Pipeline execution failed. Please check the logs."
        }
    }
}
