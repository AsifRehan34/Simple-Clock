pipeline {
    agent any

    environment {
        APP_DIR = '/var/www/clock-app'      // Current live NGINX serving directory
        NEW_APP_DIR = '/var/www/clock-app-temp'  // Temporary directory for pulling updates
        GIT_REPO_URL = 'https://github.com/AsifRehan34/Simple-Clock.git'  // GitHub repo URL
        CLOUDFRONT_DISTRIBUTION_ID = 'EZRU4ZDGNVIOS'  // Your CloudFront Distribution ID
    }

    stages {
        stage('Pull Latest Code') {
            steps {
                script {
                    // If the temporary directory exists, pull the latest updates, otherwise clone the repo
                    sh """
                        if [ -d '${NEW_APP_DIR}/.git' ]; then
                            echo 'Pulling latest changes into the temporary directory...'
                            cd ${NEW_APP_DIR} && git pull origin main
                        else
                            echo 'Cloning the repository into ${NEW_APP_DIR}...'
                            git clone ${GIT_REPO_URL} ${NEW_APP_DIR}
                        fi
                    """
                }
            }
        }

        stage('Deploy to NGINX') {
            steps {
                script {
                    // Backup the current app directory (optional, in case rollback is needed)
                    echo "Renaming current app directory for backup..."
                    sh "mv ${APP_DIR} ${APP_DIR}-backup"

                    // Rename the temp directory to the live app directory to switch over
                    sh "mv ${NEW_APP_DIR} ${APP_DIR}"

                    // Reload NGINX to apply the new updates
                    echo "Reloading NGINX..."
                    sh "sudo systemctl reload nginx"
                }
            }
        }

        stage('Invalidate CloudFront Cache') {
            steps {
                script {
                    // Invalidate CloudFront cache to serve the latest content
                    echo 'Invalidating CloudFront cache'
                    sh """
                    aws cloudfront create-invalidation --distribution-id ${CLOUDFRONT_DISTRIBUTION_ID} --paths "/*"
                    """
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
            echo 'Deployment failed. Rolling back to the previous version...'
            // Rollback if something goes wrong
            sh "mv ${APP_DIR}-backup ${APP_DIR}"
            sh "sudo systemctl reload nginx"
        }
    }
}
