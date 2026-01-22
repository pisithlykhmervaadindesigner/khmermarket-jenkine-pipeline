pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials')
        DOCKER_IMAGE_BACKEND = 'khmermarket/backend'
        DOCKER_IMAGE_FRONTEND = 'khmermarket/frontend'
        DOCKER_REGISTRY = 'docker.io'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build Backend') {
            agent {
                docker {
                    image 'maven:3.8.6-openjdk-17'
                    args '-v $HOME/.m2:/root/.m2'
                }
            }
            steps {
                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }
        
        stage('Build Frontend') {
            agent {
                docker {
                    image 'node:18'
                    args '-v $HOME/.npm:/root/.npm'
                }
            }
            steps {
                dir('fronted') {
                    sh 'npm install'
                    sh 'npm run build'
                }
            }
        }
        
        stage('Build and Push Docker Images') {
            agent any
            steps {
                script {
                    // Build and push backend image
                    withDockerRegistry([credentialsId: 'docker-hub-credentials', url: '']) {
                        sh "docker build -t ${DOCKER_IMAGE_BACKEND}:${env.BUILD_NUMBER} -t ${DOCKER_IMAGE_BACKEND}:latest ./backend/user-service"
                        sh "docker push ${DOCKER_IMAGE_BACKEND}:${env.BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE_BACKEND}:latest"
                        
                        // Build and push frontend image
                        sh "docker build -t ${DOCKER_IMAGE_FRONTEND}:${env.BUILD_NUMBER} -t ${DOCKER_IMAGE_FRONTEND}:latest ./fronted"
                        sh "docker push ${DOCKER_IMAGE_FRONTEND}:${env.BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE_FRONTEND}:latest"
                    }
                }
            }
        }
        
        stage('Deploy') {
            agent any
            steps {
                script {
                    // Update docker-compose with new image tags
                    sh """
                    sed -i 's|${DOCKER_IMAGE_BACKEND}:.*|${DOCKER_IMAGE_BACKEND}:${env.BUILD_NUMBER}|g' docker-compose/docker-compose.prod.yml
                    sed -i 's|${DOCKER_IMAGE_FRONTEND}:.*|${DOCKER_IMAGE_FRONTEND}:${env.BUILD_NUMBER}|g' docker-compose/docker-compose.prod.yml
                    """
                    
                    // Deploy using docker-compose
                    sh 'docker-compose -f docker-compose/docker-compose.prod.yml up -d'
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
            // Add notification here (e.g., Slack, email)
        }
        failure {
            echo 'Pipeline failed!'
            // Add notification here
        }
    }
}
