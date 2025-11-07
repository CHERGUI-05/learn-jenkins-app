pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh '''
                     node --version
                     npm --version
                     npm ci
                     npm run build
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                    test -f build/index.html
                    npm test
                '''
            }
        }
    }
}
