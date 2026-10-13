pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Frontend') {
            steps {
                dir('client') {
                    bat 'node -v'
                    bat 'call npm ci'
                    bat 'call npm run build'
                }
            }
        }
    }

    post {
        success { echo 'Build succeeded' }
        failure { echo 'Build failed, check the stage logs' }
    }
}
