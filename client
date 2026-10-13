pipeline {
    agent any

    environment {
        APP_HOST = 'ubuntu@<app-private-ip>'
        WEB_HOST = 'ubuntu@<web-private-ip>'
        APP_DIR  = '/home/ubuntu/<repo>/server'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Frontend') {
            steps {
                dir('client') {
                    sh 'npm ci'
                    sh 'npm run build'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                sshagent(credentials: ['deploy-key']) {
                    sh """
                        rsync -az --delete --exclude node_modules --exclude .env \
                          -e 'ssh -o StrictHostKeyChecking=no' server/ \$APP_HOST:\$APP_DIR/
                        ssh -o StrictHostKeyChecking=no \$APP_HOST '
                          cd \$APP_DIR && npm ci --omit=dev &&
                          (pm2 restart api || pm2 start server.js --name api) && pm2 save'
                    """
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                sshagent(credentials: ['deploy-key']) {
                    sh """
                        rsync -az --delete -e 'ssh -o StrictHostKeyChecking=no' \
                          client/build/ \$WEB_HOST:/var/www/html/
                    """
                }
            }
        }

        stage('Health Check') {
            steps {
                sh 'sleep 5'
                sh 'curl -f http://<web-private-ip>/api/<route>'
            }
        }
    }

    post {
        success { echo 'Deployment succeeded' }
        failure { echo 'Deployment failed, check the stage logs' }
    }
}
