pipeline {
    agent any

    environment {
        APP_HOST = 'ubuntu@<app-public-ip>'
        WEB_HOST = 'ubuntu@<web-public-ip>'
        APP_DIR  = '/home/ubuntu/react-node-mongo/server'
    }

    stages {
        stage('Build Frontend') {
            steps {
                dir('client') {
                    bat 'call npm ci'
                    bat 'call npm run build'
                }
            }
        }

        stage('Deploy Backend') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy-key', keyFileVariable: 'KEY')]) {
                    bat '''
                    icacls "%KEY%" /inheritance:r
                    icacls "%KEY%" /grant:r "%USERNAME%:R"
                    ssh -i "%KEY%" -o StrictHostKeyChecking=no %APP_HOST% "mkdir -p %APP_DIR%"
                    scp -i "%KEY%" -o StrictHostKeyChecking=no -r server\\* %APP_HOST%:%APP_DIR%/
                    ssh -i "%KEY%" -o StrictHostKeyChecking=no %APP_HOST% "cd %APP_DIR% && npm ci --omit=dev && (pm2 restart api || pm2 start server.js --name api) && pm2 save"
                    '''
                }
            }
        }

        stage('Deploy Frontend') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy-key', keyFileVariable: 'KEY')]) {
                    bat '''
                    icacls "%KEY%" /inheritance:r
                    icacls "%KEY%" /grant:r "%USERNAME%:R"
                    ssh -i "%KEY%" -o StrictHostKeyChecking=no %WEB_HOST% "rm -rf /var/www/html/*"
                    scp -i "%KEY%" -o StrictHostKeyChecking=no -r client\\build\\* %WEB_HOST%:/var/www/html/
                    '''
                }
            }
        }

        stage('Health Check') {
            steps {
                bat 'curl -f http://<web-public-ip>/api/<route>'
            }
        }
    }
}
