pipeline {
    agent any

    environment {
        NODE_ENV = 'test'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Tests') {
            steps {
                // توليد ملفات JUnit XML لتقارير Jenkins
                sh 'npm test -- --ci --reporters=default --reporters=jest-junit'
            }
        }

        stage('Archive Test Results') {
            steps {
                // مسار ملفات XML الناتجة عن jest-junit
                junit '**/test-results/*.xml'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
