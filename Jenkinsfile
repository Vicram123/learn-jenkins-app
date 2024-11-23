
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
        stage('Test'){
              agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
              }
              steps {
                  sh '''
                  test -f build/index.html
                  npm test
                  '''
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
             netlify --version
              '''
            }
        }
    }

    post {
        always {
            publishHTML(
              allowMissing: false,
              alwaysLinkToLastBuild: false,
              keepAll: false,
              reportDir: '.'
            )
            junit 'test-results/junit.xml'
        }
    }
}