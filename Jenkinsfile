pipeline {
    agent any
    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION = 'us-east-1'
    stages {
        stage ('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    aws s3 ls
                '''
                     }
                }

            }
        }
        stage('Clean Workspace') {
            steps {
                always {
                    cleanWs()
         }
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-u 1000:1000'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci --unsafe-perm
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                
                    steps {
                        sh '''
                        #test -f build/index.html
                        npm test
                        '''
                    }
                }
    
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    // args '-u root:root'
                }
            }
        
            steps {
                sh '''
                  npm install  serve
                  node_modules/.bin/serve -s build &
                  sleep 10
                  npx playwright test --reporter=html
                '''
            }
        }
            }
          
        }

/*
        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
        
            steps {
                sh '''
                   #test -f build/index.html
                   npm test
                '''
            }
        }
    
        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    // args '-u root:root'
                }
            }
        
            steps {
                sh '''
                  npm install  serve
                  node_modules/.bin/serve -s build &
                  sleep 10
                  npx playwright test --reporter=html
                '''
            }
        }
        */
    }    

    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}

