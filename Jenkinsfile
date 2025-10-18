pipeline {
    agent any

    environment {
        //NETLIFY_SITE_ID = '7d6dde62-ae7f-43e3-aff3-539061b6b105'
        //NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_DEFAULT_REGION = 'ap-southeast-1'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'
    }

    stages {
        
        /*
        stage('Docker') {
            steps {
                sh 'docker build -t my-playwright .'
            }
        }*/
        //
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo 'Small change'
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker image') {
            agent {
                docker {
                    /*2.31.18*/
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                sh '''
                    amazon-linux-extras install docker
                    docker build -t myjenkinsapp .
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    /*2.31.18*/
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            /*environment {
                //AWS_S3_BUCKET = 'learn-jenkins-20251018'
            }*/
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    // some block
                    sh '''
                        aws --version
                        yum install jq -y
                        #aws s3 sync build s3://$AWS_S3_BUCKET/
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://AWS/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
        }


        /*stage ('Run Tests') {
            parallel {
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            #echo "Test stage"
                            #test -f build/index.html
                            npm test 
                        '''
                    }
                    post {
                        always {
                            junit "jest-results/junit.xml"
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            //image 'mcr.microsoft.com/playwright:v1.46.0-jammy'
                            //image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                            reuseNode true
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }

            }
        }
        
        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to Staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    #npm run build
                    netlify deploy --dir=build --json > deploy-output.json
                    
                '''
                script {
                    env.STAGING_URL = sh(script: "node-jq -r '.deploy_url' deploy-output.json", returnStdout: true)
                }
                echo "env.STAGING_URL = ${env.STAGING_URL}"
            }
        }

        stage('Staging E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    //image 'mcr.microsoft.com/playwright:v1.46.0-jammy'
                    //image 'mcr.microsoft.com/playwright:v1.50.0-noble'
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
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Apprival') {
            steps {
                timeout(time: 15, unit: 'SECONDS') {
                    // some block
                    input message: 'Ready to deploy?', ok: 'Yes, I am sure I want to deploy!'
                }
                
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploying to Production. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    #npm run build
                    netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'my-playwright'
                    //image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    //image 'mcr.microsoft.com/playwright:v1.46.0-jammy'
                    //image 'mcr.microsoft.com/playwright:v1.50.0-noble'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://clever-sunburst-33f857.netlify.app'
            }
            steps {
                sh '''
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
        */
    }
}