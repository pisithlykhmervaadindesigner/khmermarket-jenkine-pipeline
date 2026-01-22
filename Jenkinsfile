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
        // Telegram Configuration
        TELEGRAM_BOT_TOKEN = credentials('telegram-bot-token')  // Create this credential in Jenkins
        TELEGRAM_CHAT_ID = credentials('telegram-id-khmermarket')  // Replace with your channel username
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
                    notifyTelegram("🔄 *Build Started* \n*Environment:* ${params.BUILD_ENV}\n*Status:* In Progress")
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

                        try {
                            sh """
                                echo "Logging into Docker Hub..."
                                echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin

                                echo "Building Docker image: $imageName"
                                cd $WORKSPACE/khmermarket-backend-service
                                docker build -f user-service/Dockerfile -t $imageName .

                                echo "Pushing to Docker Hub..."
                                docker push $imageName

                                docker logout
                            """
                            currentBuild.result = 'SUCCESS'
                        } catch (Exception e) {
                            currentBuild.result = 'FAILURE'
                            error "Build failed: ${e.message}"
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    try {
                        sh '''
                            ssh -o StrictHostKeyChecking=no -i /var/lib/jenkins/.ssh/id_rsa deploy@167.71.193.99 '
                                cd khmermarket-docker-compose
                                docker-compose pull
                                docker-compose up -d
                                docker image prune -f
                            '
                        '''
                        echo "Deployed successfully!"
                        currentBuild.result = 'SUCCESS'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        error "Deployment failed: ${e.message}"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Deployment completed for environment: ${params.BUILD_ENV}"
            sh 'rm -f $HOME/.docker/config.json'

            script {
                def status = currentBuild.result == 'SUCCESS' ? '✅ *SUCCESS*' : '❌ *FAILED*'
                def message = """
                    *Deployment Status*
                    *Environment:* ${params.BUILD_ENV}
                    *Status:* ${status}
                    *Build Number:* ${env.BUILD_NUMBER}
                    *Build URL:* ${env.BUILD_URL}
                """
                notifyTelegram(message)
            }
        }
    }
}

// Telegram notification function
def notifyTelegram(String message) {
    def url = "https://api.telegram.org/bot${TELEGRAM_BOT_TOKEN}/sendMessage"
    def payload = [
        chat_id: "${TELEGRAM_CHAT_ID}",
        text: message,
        parse_mode: 'Markdown',
        disable_web_page_preview: true
    ]

    def json = groovy.json.JsonOutput.toJson(payload)

    sh """
        curl -s -X POST ${url} \
        -H 'Content-Type: application/json' \
        -d '${json}'
    """
}