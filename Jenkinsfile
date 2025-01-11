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
        stage ("Run Tests") {
            parallel {
                stage ("Unit Test") {
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
                    post {
                        always {
                            junit 'junit-results/junit.xml'
                        }
                    }
                }
                stage (E2E) {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            // to run command as root -> args '-u  root:root' //
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
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