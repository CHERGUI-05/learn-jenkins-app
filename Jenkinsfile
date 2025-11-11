pipeline {
    agent any

    environment {
        NETLIFY_AUTH_TOKEN = credentials('netlify-token') // ضع هنا الـ Jenkins Credential المناسب
    }

    stages {
        stage('Checkout') {
            steps {
                // استنساخ المشروع من GitHub
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/CHERGUI-05/learn-jenkins-app.git']]
                ])
            }
        }

        stage('Build') {
            steps {
                script {
                    // تشغيل npm داخل Docker container بدون مشاكل الأذونات
                    docker.image('node:18-bullseye').inside("-v ${env.WORKSPACE}:${env.WORKSPACE} -w ${env.WORKSPACE}") {
                        sh 'npm ci'
                        sh 'npm run build'
                    }
                }
            }
        }

        stage('Tests') {
            parallel {
                stage('Unit tests') {
                    steps {
                        script {
                            docker.image('node:18-bullseye').inside("-v ${env.WORKSPACE}:${env.WORKSPACE} -w ${env.WORKSPACE}") {
                                sh 'npm test'
                                junit '**/junit.xml'  // إذا كنت تستخدم Jest مع jest-junit
                            }
                        }
                    }
                }

                stage('E2E tests') {
                    steps {
                        script {
                            docker.image('mcr.microsoft.com/playwright:v1.39.0-jammy').inside("-v ${env.WORKSPACE}:${env.WORKSPACE} -w ${env.WORKSPACE}") {
                                sh 'npx playwright test'
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
            }
            steps {
                echo 'Deployment stage (Add your deploy commands here)'
                // مثال: sh 'npx netlify deploy --prod'
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
