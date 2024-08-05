pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '6f8f71ad-d286-48b5-b7b6-0be561c76f19'
        NETLIFY_AUTH_TOKEN = 'netlify'
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
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "E2E Test stage"
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 20
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright html report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }                    
                }                
            }
        }
        stage('Deploy') {
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
                echo "Site id - $NETLIFY_SITE_ID is deployed"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }
    }
}