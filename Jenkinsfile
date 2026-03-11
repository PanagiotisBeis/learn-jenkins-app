pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '15966731-185a-446d-af5d-bf9b9cf27e8f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Tests')
        {
            parallel {
                stage ('Unit Tests'){
                    agent {
                        docker {
                            image "node:18-alpine"
                            reuseNode true
                        }
                    }
                    steps{
                        echo "Test stage"
                        sh '''
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

                stage ('E2E'){
                    agent {
                        docker {
                            image "mcr.microsoft.com/playwright:v1.49.1-noble"
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
                            npx playwright test --reporter=html
                        '''
                    }
                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }                    
                }
            }
        }
        stage('Deploy staging') {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "DEPLOY SITE TO STAGING. SITE ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --no-build --dir=build
                '''
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image "node:18-alpine"
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "DEPLOY SITE TO PRODUCTION. SITE ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --no-build --dir=build --prod
                '''
            }
        }
        stage ('Production E2E'){
            agent {
                docker {
                    image "mcr.microsoft.com/playwright:v1.49.1-noble"
                    reuseNode true
                }
            }
            environment {
                CI_ENVIRONMENT_URL = 'https://splendid-mousse-ab6c15.netlify.app'
            }
            steps{
                sh '''
                    npx playwright test --reporter=html
                '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }                    
        }
    }
}
