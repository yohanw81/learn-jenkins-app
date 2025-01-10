pipeline {
    agent any

    stages {
        stage ('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
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
                    ls -al
                '''
            }
        }
        stage('w/o docker') {
            steps {
                sh '''
                    echo "Without Docker"
                    touch container-no.txt
                    ls -la
                '''
            }
        }
        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "With Docker"
                    npm --version
                    touch container-yes.txt
                    ls -al
                '''
            }
        }
    }
}