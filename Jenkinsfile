pipeline {
    agent any
    
    parameters {
        choice(name: 'BUILD_ENV',
               choices: ['dev', 'staging', 'production'],
               description: 'Select the build environment')
    }

    stages {
        stage('Check Connection') {
            steps {
                script {
                    // Test basic connectivity
                    echo "Testing connection to GitHub..."
                    sh 'curl -I https://github.com'
                    echo "Trying to clone repository..."
                    sh '''
                        rm -rf test-repo
                        git clone --depth 1 https://github.com/pisithlykhmervaadindesigner/khmermarket-docker-compose.git test-repo || echo "Clone failed"
                        ls -la test-repo 2>/dev/null || echo "No repository was cloned"
                    '''
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