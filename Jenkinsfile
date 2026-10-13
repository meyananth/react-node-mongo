pipeline {
    agent any

    tools { nodejs 'node20' }

    stages {
        stage('Debug') {
            steps {
                bat 'dir'
                bat 'dir /s /b package.json | findstr /v node_modules'
            }
        }
    }
}
