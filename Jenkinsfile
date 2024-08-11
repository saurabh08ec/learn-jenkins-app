pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '6f8f71ad-d286-48b5-b7b6-0be561c76f19'
        NETLIFY_AUTH_TOKEN = 'nfp_GNYQXdngbumW2aqhxz2AjzVEf4uooorTc172'
    }
    stages {
        stage('build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                echo "Building Jenkins App"
                sh '''
                    echo "Small change added"
                    ls -ltra
                    node --version
                    npm --version
                    npm ci
                    npm run build 
                    ls -ltra
                '''
            }
        }
        stage ('testing')
        {
            parallel ('All test')
            {
                stage('test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "Test stage"
                        sh '''
                            ls ./build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }
                stage('E2E-Local') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "E2E-Local Test stage"
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 20
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright html report-Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }                    
                }                
            }
        }
        stage('Deploy-Prod-staging') {
            agent {
                docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
            steps {
                echo "Deployment stage"
                sh '''
                npm install netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to staging, Site id - $NETLIFY_SITE_ID is deployed"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-stage.json
                '''
            script {
                    env.my-site = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' deploy-stage.json", returnStdout: true)
                } 
                echo "Staging URL is ${env.my-site}"
            }
        }        
        stage('Approval') {
            steps {
                timeout(1) {
                    input message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }        
        stage('Deploy-Production') {
            agent {
                docker {
                        image 'node:18-alpine'
                        reuseNode true
                    }
                }
            steps {
                echo "Deployment stage"
                sh '''
                npm install netlify-cli
                node_modules/.bin/netlify --version
                echo "Deploying to Production, Site id - $NETLIFY_SITE_ID is deployed"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
        stage('E2E-Production-stage') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://merry-pie-15b3fa.netlify.app'
            }                        
            steps {
                echo "E2E-Production Test stage"
                sh '''
                npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright html report-Production', reportTitles: '', useWrapperFileDirectly: true])
                }
            }            
        }    
    }
}