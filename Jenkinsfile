pipeline {
    agent any

    environment{
            APP_NAME = 'learn CI/CD with Jenkins'
            BUILD_NUMBER = "${env.BUILD_NUMBER}"
            IMAGE_VERSION = "v_${BUILD_NUMBER}"
            NETLIFY_SITE_ID = 'f28b3e7b-7f7c-4e11-9db5-3ab4f79ec48b'
            NETLIFY_AUTH_TOKEN = credentials('netlify-token')                     
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
                    ls -la
                    node --version                    
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''               
            }
        }

        stage('Run Test'){
            parallel{
                /*
                stage('SonarQube analysis') {
                    steps {
                        sh "/usr/bin/sonar-scanner"
                    }
                } */
                stage('Test'){
                    agent{
                        docker{
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps{ 
                        sh '''
                        echo "Test Stage"
                        find  ./build -name index.html
                        test -f ./build/index.html
                        npm test
                        '''
                    }
                    post {        
                        always{
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E'){
                    agent{
                         docker{
                            image 'mcr.microsoft.com/playwright:v1.46.0-jammy'
                            reuseNode true
                        }
                    }
                    steps{
                    sh '''
                        npx playwright install                               
                        npm install serve
                        node_modules/.bin/serve -s build &
                        sleep 10
                        npx playwright test --reporter=html
                    '''
                    }
                    post {
                        always{
                        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy'){
            agent{
                docker{
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    node_modules/.bin/netlify --version
                    echo "deploying to product. Site id  : $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''               
            }
        }

        stage('Prod E2E'){
        agent{
                docker{
                image 'mcr.microsoft.com/playwright:v1.46.0-jammy'
                reuseNode true
            }
        }

        environment{
            CI_ENVIRONMENT_URL = 'https://cool-croissant-3a12c9.netlify.app'
        }

        steps{
        sh '''
            npx playwright test --reporter=html
        '''
        }
        post {
            always{
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Report', reportTitles: '', useWrapperFileDirectly: true])
            }
        }
    }

    }
}