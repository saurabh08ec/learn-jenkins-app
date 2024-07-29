pipeline {
    agent any

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
        stage('test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls ./build/index.html
                    npm test
                '''
            }
        }
    }
}