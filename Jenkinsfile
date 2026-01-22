pipeline {
    agent any
    
    parameters {
        choice(name: 'BUILD_ENV',
               choices: ['dev', 'staging', 'production'],
               description: 'Select the build environment')
    }

    environment {
        DOCKER_USERNAME_CREDENTIALS_ID = 'docker-hub-username'
        DOCKER_PASSWORD_CREDENTIALS_ID = 'docker-hub-password'
        WORKSPACE = '/var/lib/jenkins/workspace/khmermarket/khmermarket-pipeline'
    }

    stages {
        stage('Clone Repository') {
            steps {
                script {
                    echo "Cloning repository..."
                    sh '''
                        cd $WORKSPACE
                        rm -rf khmermarket-backend-service
                        git clone https://github.com/pisithlykhmervaadindesigner/khmermarket-backend-service.git
                        echo "Repository cloned successfully!"
                    '''
                }
            }
        }

        stage('Build and Push') {
            steps {
                script {
                    withCredentials([
                        string(credentialsId: env.DOCKER_USERNAME_CREDENTIALS_ID, variable: 'DOCKER_USER'),
                        string(credentialsId: env.DOCKER_PASSWORD_CREDENTIALS_ID, variable: 'DOCKER_PASS')
                    ]) {
                        def imageName = "${DOCKER_USER}/backend-user-service:latest"

                        sh """
                            echo "Logging into Docker Hub..."
                            echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                            echo "Building Docker image: $imageName"
                            cd $WORKSPACE/khmermarket-backend-service
                            docker build -f user-service/Dockerfile -t $imageName .

                            echo "Pushing to Docker Hub..."
                            docker push $imageName

                            echo "Logging out from Docker Hub"
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Deploying to server..."
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa deploy@167.71.193.99 '

                            cd khmermarket-docker-compose

                            docker-compose pull

                            docker-compose up -d

                            docker image prune -f
                         '
                         echo "Deployed successfully!"
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Deployment completed for environment: ${params.BUILD_ENV}"
            sh 'rm -f $HOME/.docker/config.json'
        }
        success {
            echo "✅ Pipeline succeeded!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
    }
}