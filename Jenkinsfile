pipeline {
    agent any

    stages {
        // This is a single line comment
        /*
        This
        is
        a multi line block comment
        you can use the block comments to disable a stage
        */
        /*stage ('Build') {
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
        */
        stage ("Test") {
            agent{
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps{
            echo 'Running the Test Stage...'
            sh '''
                test -f build/index.html
                # in shell - usual # can be used to comment
                npm test
            '''
            }
        }
        stage (E2E) {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.49.1-noble'
                }
            }
            steps {
                sh '''
                    npm install -g serve
                    serve -s build
                    npx playwright test
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
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}