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
                dir('frontend') {
                    bat 'call npm ci'
                    bat 'call npm run build'
                }
            }
        }

        stage('Check Backend') {
            steps {
                dir('backend') {
                    bat 'call npm ci'
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
