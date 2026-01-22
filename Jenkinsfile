pipeline {
    agent any
    
    parameters {
        choice(name: 'BUILD_ENV', choices: ['dev', 'staging', 'production'], description: 'Select the build environment')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
                // List the directory structure for debugging
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building for ${params.BUILD_ENV} environment..."

                    // Check if backend directory exists
                    if (fileExists('backend')) {
                        echo 'Building backend...'
                        sh 'cd backend && mvn clean package -DskipTests'
                    } else {
                        echo 'No backend directory found, skipping backend build'
                    }

                    // Check if frontend directory exists
                    if (fileExists('fronted')) {
                        echo 'Building frontend...'
                        sh 'cd fronted && npm install'
                        sh 'cd fronted && npm run build'
                    } else {
                        echo 'No frontend directory found, skipping frontend build'
                    }
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
            cleanWs()
        }
    }
}