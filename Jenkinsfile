pipeline {
    agent any
    enviroment{
        NETLFIY_PROJ_ID = 'b010c224-fcda-4f8c-a068-3c3b6f63e8f2'
        NETLFIY_AUTH_Token =  credentials('NETIFY-TOKEN')
    }

    stages {
        stage('Build') {
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                ls -ls
                node --version
                npm --version 
                npm ci
                npm run build 
                ls -la
                '''  
            }
        }
        stage('Tests') {

            parallel{

                stage('Unit Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{
                        sh '''
                        echo ''
                        find build -type f -iname "index.html"
                        npm test
                        '''

                    }
                    post{
                        always{
                            junit 'test-results/junit.xml'
                        }
                    }

                }
                stage('E2E Test'){
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
                            npx playwright test  --reporter=html
                        '''
                    }
                    post{
                        always{
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                    

                }



            }
            
        }
        
        
    }
    stage('Deploy') {
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
                        echo "Deploying to production. Site ID: $NETLFIY_PROJ_IDde"
                        node_modules/.bin/netlify status
                        node_modules/.bin/netlify deploy --dir=build --prod
                    '''
                }
            }
        }



}