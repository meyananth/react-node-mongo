pipeline {
    agent any

    tools { nodejs 'node20' }

    stages {
        stage('Build Frontend') {
            steps {
                dir('client') {
                    bat 'node -v'
                    bat 'call npm install'
                    bat 'call npm run build'
                }
            }
        }
    }
}
