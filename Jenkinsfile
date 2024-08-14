pipeline {
    agent any

    environment{
            APP_NAME = 'learn CI/CD with Jenkins'
            BUILD_NUMBER = "${env.BUILD_NUMBER}"
            IMAGE_VERSION = "v_${BUILD_NUMBER}"

    }

    parameters{
        string(defaultValue: "develop", description: 'Branch Specifier', name: 'SPECIFIER')
        booleanParam(defaultValue: false, description: 'Deploy to QA Environment ?', name: 'DEPLOY_QA')
        booleanParam(defaultValue: false, description: 'Deploy to UAT Environment ?', name: 'DEPLOY_UAT')
        booleanParam(defaultValue: false, description: 'Deploy to PROD Environment ?', name: 'DEPLOY_PROD')
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
                stage('SonarQube analysis') {
                    steps {
                        sh "/usr/bin/sonar-scanner"
                    }
                }
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
                    agent{ docker{
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
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
                }
            }
        }
    }
}
