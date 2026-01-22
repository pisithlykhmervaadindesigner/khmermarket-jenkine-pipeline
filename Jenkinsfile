pipeline {
    agent any
    
    parameters {
        choice(name: 'BUILD_ENV', choices: ['dev', 'staging', 'production'],
               description: 'Select the build environment')
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                // Use the credentials ID you created in Jenkins
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building for ${params.BUILD_ENV} environment..."
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