pipeline {
    agent any
    environment {
        APP_DIR = '/var/www/clock-app'  // NGINX serving directory
        GIT_REPO_URL = 'https://github.com/AsifRehan34/Simple-Clock.git'  // GitHub repo URL
    }
    stages {
        stage('Clone or Pull Repository') {
            steps {
                script {
                    // Check if directory exists, if yes pull updates
                    sh """
                    if [ -d "${APP_DIR}/.git" ]; then
                        cd ${APP_DIR} && git pull origin main
                    else
                        git clone ${GIT_REPO_URL} ${APP_DIR}
                    fi
                    """
                }
            }
        }
        stage('Deploy to NGINX') {
            steps {
                script {
                    // Ensure NGINX files are updated
                    sh "cp -r * ${APP_DIR}/"
                    
                    // Restart NGINX to apply changes
                    sh "sudo systemctl reload nginx"
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment was successful.'
        }
        failure {
            echo 'Deployment failed.'
        }
    }
}
