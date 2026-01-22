pipeline {
    agent any
    
    parameters {
        choice(name: 'BUILD_ENV',
               choices: ['dev', 'staging', 'production'],
               description: 'Select the build environment')
    }

    environment {
        // Reference your existing credentials by their IDs
        DOCKER_USERNAME_CREDENTIALS_ID = 'docker-hub-username'  // Your username secret text credential ID
        DOCKER_PASSWORD_CREDENTIALS_ID = 'docker-hub-password'  // Your password secret text credential ID
    }

    stage('Clone Repository') {
                steps {
                    script {
                        echo "Cloning repository..."
                        sh '''
                            cd /var/lib/jenkins/workspace/khmermarket/khmermarket-pipeline

                            rm -rf khmermarket-backend-service

                            git clone https://github.com/pisithlykhmervaadindesigner/khmermarket-backend-service.git

                            echo "Repository cloned successfully!"

                            cd khmermarket-backend-service

                            ls -la  # This will show the repository contents
                        '''
                    }
                }
            }

    stages {
        stage('Build and Push') {
            steps {
                script {
                    // Get both credentials
                    withCredentials([
                        string(credentialsId: env.DOCKER_USERNAME_CREDENTIALS_ID, variable: 'DOCKER_USER'),
                        string(credentialsId: env.DOCKER_PASSWORD_CREDENTIALS_ID, variable: 'DOCKER_PASS')
                    ]) {
                        // Set the image name with the specified format
                        def imageName = "${DOCKER_USER}/backend-user-service:latest"

                        sh """
                            echo "Logging into Docker Hub..."
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                            echo "Building Docker image: $imageName"
                            docker build -f user-service/Dockerfile -t $imageName .

                            echo "Pushing to Docker Hub..."
                            docker push $imageName

                            echo "Logging out from Docker Hub"
                            docker logout

                            echo "Successfully pushed $imageName"
                        """
                    }
                }
            }
        }
    }
}