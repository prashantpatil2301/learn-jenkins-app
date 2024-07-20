pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '94483fb4-93e1-42e0-b0e8-16f9ffc64d0d'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        CI_ENVIRONMENT_URL = ''
    }

    stages {
        
        stage('Build') {
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
                    ls -la
                '''
            }
        }
        
        stage('Tests') {
            parallel {
                    stage('Test') {
                        agent {
                            docker {
                                image 'node:18-alpine'
                                reuseNode true
                            }
                        }

                        steps {
                            sh '''
                                echo 'for polling this change is introduced'
                                #test -f build/index.html
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
                            sh '''
                                npm install serve
                                node_modules/.bin/serve -s build &
                                sleep 10
                                npx playwright test --reporter=html
                                npx playwright test
                            '''
                        }
                        post {
                            always {
                                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'PW HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                            }
                        }
                    }
            }
        }
        

        stage('Staging Deploy') {   

            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            environment {
                            CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_DEFINED'
                    }

            steps {
                sh '''
                    npm install netlify-cli node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploying to prod site: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    CI_ENVIRONMENT_URL = $(node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json)
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Approval'){
            steps{
                timeout(time: 1, unit: 'MINUTES') {
                input message: 'Sure about Deploy?', ok: 'Yes, We need to deploy'
                }    
            }
        }

        stage('Deploy') {
            environment {
                            CI_ENVIRONMENT_URL = 'https://musical-queijadas-7f7ff4.netlify.app'
                    }
                    
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "Deploying to prod site: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

    }

    
}
