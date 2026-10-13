pipeline {
    agent any

    tools { nodejs 'node20' }

    environment {
        EC2_HOST = credentials('ec2-host')   // Secret text credential
        SSH_OPTS = '-o StrictHostKeyChecking=no -o ConnectTimeout=15'
    }

    stages {
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    bat 'npm ci || npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Check Backend') {
            steps {
                dir('backend') {
                    bat 'npm ci || npm install'
                }
            }
        }

                stage('Docker Build') {
            steps {
                bat 'docker --version'
                bat 'docker build -t react-node-frontend ./frontend'
                bat 'docker build -t react-node-backend ./backend'
            }
        }

      stage('Docker Build Check') {
    steps {
        bat 'docker build -t react-node-backend ./backend'
        bat 'docker build -t react-node-frontend ./frontend'
    }
}

        stage('Health Check') {
            steps {
                bat '''
for /L %%i in (1,1,18) do (
  curl -sf --max-time 5 http://%EC2_HOST%:3001 >nul && exit /b 0
  ping -n 11 127.0.0.1 >nul
)
exit /b 1
'''
            }
        }
    }

   post {
    success { echo 'Deployment succeeded' }
    failure { echo 'Deployment failed, check the stage logs' }
}
}
