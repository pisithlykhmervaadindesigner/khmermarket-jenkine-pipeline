pipeline {
    agent any
    
    parameters {
        choice(name: 'BUILD_ENV',
               choices: ['dev', 'staging', 'production'],
               description: 'Select the build environment')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                // For public repository
                git url: 'https://github.com/pisithlykhmervaadindesigner/khmermarket-docker-compose.git',
                    branch: 'main'

                // Show repository contents for debugging
                sh 'ls -la'
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building for ${params.BUILD_ENV} environment..."
                    // Add your build commands here
                    sh 'echo "This is a simple build step"'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed!'
            cleanWs()
        }
    }
}