pipeline {
    agent any

    environment {
        APP_HOST = '172.31.45.167'
        WEB_HOST = '172.31.36.214'
        APP_DIR  = 'https://github.com/meyananth/react-node-mongo.git
    }

    stages {
        stage('Build Frontend') {
            steps {
                dir('client') {
                    sh 'npm ci && npm run build'
                }
            }
        }

        stage('Deploy Backend') {
         pipeline {
    agent any

    environment {
        APP_HOST = 'ubuntu@<app-private-ip>'
        WEB_HOST = 'ubuntu@<web-private-ip>'
        APP_DIR  = '/home/ubuntu/<repo>/server'
    }

    stages {
        stage('Build Frontend') {
            steps {
                dir('client') {
                    sh 'npm ci && npm run build'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                sshagent(credentials: ['deploy-key']) {
                    sh 'rsync -az --delete --exclude node_modules --exclude .env -e "ssh -o StrictHostKeyChecking=no" server/ $APP_HOST:$APP_DIR/'
                    sh 'ssh -o StrictHostKeyChecking=no $APP_HOST "cd $APP_DIR && npm ci --omit=dev && (pm2 restart api || pm2 start server.js --name api)"'
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                sshagent(credentials: ['deploy-key']) {
                    sh 'rsync -az --delete -e "ssh -o StrictHostKeyChecking=no" client/build/ $WEB_HOST:/var/www/html/'
                }
            }
        }
    }
}   steps {
                sshagent(credentials: ['deploy-key']) {
                    sh 'rsync -az --delete --exclude node_modules --exclude .env -e "ssh -o StrictHostKeyChecking=no" server/ $APP_HOST:$APP_DIR/'
                    sh 'ssh -o StrictHostKeyChecking=no $APP_HOST "cd $APP_DIR && npm ci --omit=dev && (pm2 restart api || pm2 start server.js --name api)"'
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                sshagent(credentials: ['deploy-key']) {
                    sh 'rsync -az --delete -e "ssh -o StrictHostKeyChecking=no" client/build/ $WEB_HOST:/var/www/html/'
                }
            }
        }
    }
}
