pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '5738d3e7-9e4b-49bc-8da0-c3b44d8cc8be'
        NETLIFY_AUTH_TOKEN = credentials('netifly-token-id')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'learnjenkinsapp'
        AWS_DEFAULT_REGION = 'ap-southeast-1'
        AWS_DOCKER_REGISTRY = '886109032018.dkr.ecr.ap-southeast-1.amazonaws.com'
        AWS_ECS_CLUSTER = 'Jenkins-Cluster-Prod'
        AWS_ECS_SERVICE_PROD = 'lernJenkinsApp-TaskDef-SRV-Prod'
        AWS_ECS_TASK_DEF = 'LearJenkinsApp-TaskDefinition-Prod'
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
            environment {
                AWS_S3_BUCKET = 'learn-jenkins-202501211400'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-s3-access', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    echo "Hello S3" > hello.html
                    aws s3 cp hello.html s3://$AWS_S3_BUCKET/hello.html
                    aws s3 ls
                    '''
                }
                
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
        stage ('Build Docker Image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-s3-access', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                    aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY 
                    docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                '''
                }
            }
        }
        stage ('AWS - Deploy to AWS S3') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            environment {
                AWS_S3_BUCKET = 'learn-jenkins-202501211400'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-s3-access', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    aws --version
                    aws s3 sync build s3://$AWS_S3_BUCKET
                    aws s3 ls
                    '''
                }
                
            }
        }
        stage ('AWS - Deploy to AWS Elastic Cluster') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-s3-access', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                    set -e
                    aws --version
                    LATEST_TASK_DEF_REV=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                    echo $LATEST_TASK_DEF_REV
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TASK_DEF:$LATEST_TASK_DEF_REV
                    aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
                
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
                            //to run command as root -> args '-u  root:root' //
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