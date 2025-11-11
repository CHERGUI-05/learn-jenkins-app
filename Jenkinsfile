pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'fdcd0741-1aa5-4c43-adc6-3c74625e481b'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-bullseye'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh 'npm test'
                    }
                    post {
                        always {
                            junit 'test-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {  // <-- الآن داخل stages
            agent {
                docker {
                    image 'node:18-bullseye'
                    args '-u 1000:1000 -v /var/jenkins_home/workspace/learn-jenkins-app:/var/jenkins_home/workspace/learn-jenkins-app'
                    reuseNode true
                }
            }
            steps {
                dir('/var/jenkins_home/workspace/learn-jenkins-app') {
                    sh 'ls -la'
                    sh 'node --version'
                    sh 'npm --version'
                    sh 'npm install netlify-cli --no-save'
                    sh 'npx netlify status'
                    sh 'npx netlify deploy --dir=build --prod'
                }
            }
        }
    }
}
