pipeline {
    agent any
    
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
                stage('Unit tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    
                    steps {
                        sh '''
                            #test -f build/index.html
                            
                            # Vérifier l'existence du répertoire pour les rapports
                            mkdir -p jest-results
                            
                            # Exécuter Jest avec l'option de reporter JUnit
                            npm test -- --reporters=default --reporters=jest-junit
                        '''
                    }
                    
                    post {
                        always {
                            // Vérifier si les fichiers existent avant de les traiter
                            sh 'ls -la jest-results || echo "Le répertoire jest-results est vide ou n\'existe pas"'
                            junit allowEmptyResults: true, testResults: 'jest-results/junit.xml'
                        }
                    }
                }
                
                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.52.0-jammy'
                            reuseNode true
                        }
                    }
                    
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 15
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

        stage('Deploy') {
            agent {
                docker {
                    image 'node:&8-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1 -g
                    node_modules/.bin/netlify --version
                '''
            }
        }
    }
}