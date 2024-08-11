pipeline {
    agent any
    environment {
        NETLIFY_SITE_ID = '6f8f71ad-d286-48b5-b7b6-0be561c76f19'
        NETLIFY_AUTH_TOKEN = 'nfp_GNYQXdngbumW2aqhxz2AjzVEf4uooorTc172'
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
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
                stage('Docker') {
                    sh '''
                    docker build . -t advance-playwright
                    '''
                }
                stage('E2E-Local') {
                    agent {
                        docker {
                            image 'advance-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        echo "E2E-Local Test stage"
                        sh '''
                            serve -s build &
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
                yarn add netlify-cli node-jq
                node_modules/.bin/netlify --version
                echo "Deploying to staging, Site id - $NETLIFY_SITE_ID is deployed"
                node_modules/.bin/netlify status
                node_modules/.bin/netlify deploy --dir=build --json > deploy-stage.json
                sed 's/"//g' deploy-stage.json > temp.txt && cat temp.txt > deploy-stage.json
                '''
            script {
                    env.mystagingsite = sh(script: "grep 'deploy_url' deploy-stage.json |cut -d: -f2,3|cut -d , -f 1|sed 's/ //g'", returnStdout: true)
                } 
                echo "Staging URL is ${env.mystagingsite}"
            }
        }
        stage('E2E-Test-on-staging') {
            agent {
                docker {
                    image 'advance-playwright'
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = "${env.mystagingsite}"
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
    }
}