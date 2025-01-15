pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '5738d3e7-9e4b-49bc-8da0-c3b44d8cc8be'
        NETLIFY_AUTH_TOKEN = credentials('netifly-token-id')
    }

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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }
        stage ('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify link --id 5738d3e7-9e4b-49bc-8da0-c3b44d8cc8be
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --json > build-output.json
                    echo "Git Poling Test"
                '''
            }
            script {
                env.STAGING_URL = sh(script: "node_modules/.bin/node-jq -r '.deploy_url' build-output.json", returnStdout: true)
            }
        }
        stage ("Staging E2E") {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
             }
             environment {
                CI_ENVIRONMENT_URL = "${env.STAGING_URL}"
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'Staging-report', reportFiles: 'index.html', reportName: 'Playwright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage('Approval for Production Upgrade') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                input message: 'Ready to Deploy to Production', ok: 'Yes, I am sure!'
                }  
            }
        }
        stage ('Deploy Production') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify link --id 5738d3e7-9e4b-49bc-8da0-c3b44d8cc8be
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify deploy --dir=build --prod
                    echo "Git Poling Test"
                '''
            }
        }
        stage ("Prod E2E") {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
             }
             environment {
                CI_ENVIRONMENT_URL = 'https://grand-eclair-f65c7d.netlify.app'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'Prod-report', reportFiles: 'index.html', reportName: 'Playwright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
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