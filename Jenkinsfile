pipeline {
    agent any
    
    parameters {
        choice(name: 'BUILD_ENV',
               choices: ['dev', 'staging', 'production'],
               description: 'Select the build environment')
    }

    stages {
        stage('Check GitHub Connection') {
            steps {
                script {
                    echo "Testing connection to GitHub..."
                    sh 'curl -I https://github.com'
                }
            }
        }

        stage('Clone Repository') {
            steps {
                script {
                    echo "Cloning repository..."
                    sh '''
                        rm -rf test-repo
                        git clone --depth 1 https://github.com/pisithlykhmervaadindesigner/khmermarket-docker-compose.git test-repo
                        echo "Repository cloned successfully!"
                    '''
                }
            }
        }

        stage('List Repository Contents') {
            steps {
                script {
                    echo "Repository contents:"
                    sh '''
                        ls -la test-repo
                    '''
                    // Access the parameter outside of the shell script
                    echo "Environment: ${params.BUILD_ENV}"
                }
            }
        }
    }

    post {
        always {
            echo "Build completed for environment: ${params.BUILD_ENV}"
            // Uncomment the next line if you want to clean the workspace
            // cleanWs()
        }
        success {
            echo "✅ Pipeline succeeded!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}