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

        stage('Deploy to EC2') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'deploy-key', keyFileVariable: 'KEY')]) {
                    bat '''
for /f "delims=" %%u in ('whoami') do icacls "%KEY%" /inheritance:r /grant:r "%%u:R"
tar -cf app.tar --exclude=node_modules --exclude=build --exclude=.git backend frontend docker-compose.yml
scp -i "%KEY%" %SSH_OPTS% app.tar ubuntu@%EC2_HOST%:/home/ubuntu/app.tar
ssh -i "%KEY%" %SSH_OPTS% ubuntu@%EC2_HOST% "mkdir -p ~/app && tar -xf ~/app.tar -C ~/app && cd ~/app && docker compose down --remove-orphans; docker compose up -d --build"
'''
                }
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
        always  { bat 'del /q app.tar 2>nul' }
        success { echo 'Deployment succeeded' }
        failure { echo 'Deployment failed, check the stage logs' }
    }
}
