pipeline {
    agent any
    
    parameters {
        // Basic build parameters
        choice(name: 'BUILD_ENV', choices: ['dev', 'staging', 'production'], description: 'Select the build environment')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests during build')
    }

    environment {
        // Environment variables
        DOCKER_IMAGE_BACKEND = "khmermarket/backend"
        DOCKER_IMAGE_FRONTEND = "khmermarket/frontend"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                script {
                    echo "Building backend for ${params.BUILD_ENV} environment..."
                    sh 'cd backend && mvn clean package -DskipTests'
                }
            }
        }

        stage('Test Backend') {
            when {
                expression { params.RUN_TESTS == true }
            }
            steps {
                echo 'Running backend tests...'
                sh 'cd backend && mvn test'
            }
        }

        stage('Build Frontend') {
            steps {
                echo 'Installing frontend dependencies...'
                sh 'cd fronted && npm install'

                echo 'Building frontend...'
                sh 'cd fronted && npm run build'
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    def imageTag = "${params.BUILD_ENV}-${env.BUILD_NUMBER}"

                    echo "Building Docker images with tag: ${imageTag}"

                    // Build backend image
                    sh """
                    docker build -t ${DOCKER_IMAGE_BACKEND}:${imageTag} ./backend/user-service
                    """

                    // Build frontend image
                    sh """
                    docker build -t ${DOCKER_IMAGE_FRONTEND}:${imageTag} ./fronted
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        cleanup {
            // Clean up workspace after build
            cleanWs()
        }
    }
}