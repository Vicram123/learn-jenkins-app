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
    environment {
        NETLIFY_SITE_ID = credentials('ceabc61a-f40d-44b1-a80b-c7c18c50ede5')
        NETLIFY_AUTH_TOKEN = credentials('nfp_GsbbHHGhFMmTJAe7JTVEZ7ddKUrzMpTz4e51')
    }
    steps {
        sh 'npm install netlify-cli -g'
        sh 'which netlify'
        sh 'netlify login --auth-token=$NETLIFY_AUTH_TOKEN'
        sh 'netlify status'
        sh 'netlify deploy --dir=build --prod --site-id=$NETLIFY_SITE_ID'
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
