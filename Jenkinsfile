pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') // اسم الـ credential في Jenkins
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'node:18-bullseye'
                    args '-u 1000:1000'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "Node version:"
                    node --version
                    echo "NPM version:"
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-bullseye'
                            args '-u 1000:1000'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm test
                        '''
                        junit '**/test-results/*.xml'
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            args '-u 1000:1000'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install -g serve
                            serve -s build & sleep 10
                            npx playwright test --reporter=html
                        '''
                        publishHTML(target: [
                            reportDir: 'playwright-report',
                            reportFiles: 'index.html',
                            reportName: 'Playwright HTML Report'
                        ])
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-bullseye'
                    args '-u 1000:1000'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli --no-save
                    npx netlify deploy --dir=build --prod
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
