pipeline {
    agent any

    tools { nodejs 'node20' }

    environment {
        PATH        = "C:\\Program Files\\Amazon\\AWSCLIV2;${env.PATH}"
        REPO_URL    = 'https://github.com/meyananth/react-node-mongo.git'
        APP_DIR     = 'react-node-mongo'
        INSTANCE_ID = 'i-0a74867d3eaf7a9d9'
        AWS_REGION  = 'eu-north-1'
        KEY_FILE    = 'C:/keys/deploy-key.pem'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: "${REPO_URL}"
            }
        }

        stage('Install Dependencies') {
            parallel {
                stage('Backend') {
                    steps {
                        dir('backend') {
                            bat 'npm install'
                        }
                    }
                }
                stage('Frontend') {
                    steps {
                        dir('frontend') {
                            bat 'npm install'
                        }
                    }
                }
            }
        }

        stage('Prepare EC2') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                                  credentialsId: 'aws-creds',
                                  accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                                  secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                    // Start instance if stopped and wait until ready
                    bat '''
                    aws ec2 start-instances --instance-ids %INSTANCE_ID% --region %AWS_REGION%
                    aws ec2 wait instance-running --instance-ids %INSTANCE_ID% --region %AWS_REGION%
                    aws ec2 wait instance-status-ok --instance-ids %INSTANCE_ID% --region %AWS_REGION%
                    '''

                    // Get current public IP
                    script {
                        env.EC2_HOST = bat(returnStdout: true, script: '@aws ec2 describe-instances --instance-ids %INSTANCE_ID% --region %AWS_REGION% --query "Reservations[0].Instances[0].PublicIpAddress" --output text').trim()
                        echo "EC2 Public IP: ${env.EC2_HOST}"
                    }

                    // Allow this Jenkins machine's IP on port 22 (ignore "already exists")
                    bat '''
                    @echo off
                    for /f %%i in ('curl.exe -s https://checkip.amazonaws.com') do set MYIP=%%i
                    for /f %%g in ('aws ec2 describe-instances --instance-ids %INSTANCE_ID% --region %AWS_REGION% --query "Reservations[0].Instances[0].SecurityGroups[0].GroupId" --output text') do set SGID=%%g
                    aws ec2 authorize-security-group-ingress --group-id %SGID% --protocol tcp --port 22 --cidr %MYIP%/32 --region %AWS_REGION% || echo Rule already exists
                    exit /b 0
                    '''
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                bat '''
                ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=NUL -o ConnectTimeout=20 -i "%KEY_FILE%" ubuntu@%EC2_HOST% "command -v docker >/dev/null 2>&1 || (curl -fsSL https://get.docker.com | sudo sh); if [ ! -d %APP_DIR% ]; then git clone %REPO_URL%; fi && cd %APP_DIR% && git pull origin main && sudo docker compose up -d --build && sudo docker image prune -f"
                '''
            }
        }

        stage('Health Check') {
            steps {
                retry(18) {
                    sleep time: 10, unit: 'SECONDS'
                    bat 'curl.exe -s -o NUL --max-time 10 http://%EC2_HOST%:3000'
                }
                retry(6) {
                    sleep time: 5, unit: 'SECONDS'
                    bat 'curl.exe -s -o NUL --max-time 10 http://%EC2_HOST%:5000'
                }
            }
        }
    }

    post {
        success { echo "Deployed: http://${env.EC2_HOST}:3000" }
        failure { echo 'Deployment failed, check console output' }
    }
}
