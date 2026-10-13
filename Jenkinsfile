pipeline {
    agent any

    environment {
        APP_HOST = '172.31.45.167'
        WEB_HOST = '172.31.36.214'
        APP_DIR  = '/home/ubuntu/react-node-mongo/server'
        SSH_OPTS = '-o StrictHostKeyChecking=no'
    }

    stages {
        stage('Build Frontend') {
            steps {
                dir('client') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy-key', keyFileVariable: 'KEY')]) {
                    bat '''
                        tar -cf server.tar --exclude=node_modules --exclude=.env -C server .
                        scp -i "%KEY%" %SSH_OPTS% server.tar ubuntu@%APP_HOST%:/tmp/server.tar
                        ssh -i "%KEY%" %SSH_OPTS% ubuntu@%APP_HOST% "mkdir -p %APP_DIR% && cd %APP_DIR% && tar -xf /tmp/server.tar && rm /tmp/server.tar && npm install --omit=dev && (pm2 restart api || pm2 start server.js --name api) && pm2 save"
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy-key', keyFileVariable: 'KEY')]) {
                    bat '''
                        tar -cf web.tar -C client/build .
                        scp -i "%KEY%" %SSH_OPTS% web.tar ubuntu@%WEB_HOST%:/tmp/web.tar
                        ssh -i "%KEY%" %SSH_OPTS% ubuntu@%WEB_HOST% "rm -rf /var/www/html/* && tar -xf /tmp/web.tar -C /var/www/html && rm /tmp/web.tar"
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                bat 'powershell -Command "Start-Sleep -Seconds 5"'
                bat 'curl.exe -f http://%WEB_HOST%/api/YOUR_ROUTE'
            }
        }
    }

    post {
        success { echo 'Deployment succeeded' }
        failure { echo 'Deployment failed, check the stage logs' }
    }
}
