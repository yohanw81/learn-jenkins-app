pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '5738d3e7-9e4b-49bc-8da0-c3b44d8cc8be'
        NETLIFY_AUTH_TOKEN = credentials('netifly-token-id')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }

    stages {
        // This is a single line comment
        /*
        This
        is
        a multi line block comment
        you can use the block comments to disable a stage
        */
        stage ('AWS') {
            agent {
                docker {
                    /*image 'amazon/aws-cli' -- for the latest amazon cocket image*/
                    image 'amazon/aws-cli:2.23.2' /* for a specific version */
                    args "--entrypoint=''"
                }
            }
            steps {
                sh '''
                    aws --version
                '''
            }

        }
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
                            image 'my-playwright'
                            reuseNode true
                            // to run command as root -> args '-u  root:root' //
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
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
        stage ("Deploy Staging") {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
             }
             environment {
                CI_ENVIRONMENT_URL = "STAGING_URL_TO_BE_SET"
            }
            steps {
                sh '''
                    netlify --version
                    netlify status
                    netlify link --id 5738d3e7-9e4b-49bc-8da0-c3b44d8cc8be
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify deploy --dir=build --json > build-output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' build-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        stage ("Prod Deployement") {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
             }
             environment {
                CI_ENVIRONMENT_URL = 'https://grand-eclair-f65c7d.netlify.app'
            }
            steps {
                sh '''
                    node --version
                    netlify --version
                    netlify status
                    netlify link --id 5738d3e7-9e4b-49bc-8da0-c3b44d8cc8be
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod Report', reportTitles: '', useWrapperFileDirectly: true])
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