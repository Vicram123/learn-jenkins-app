
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
        stage('E2E') {
  agent {
    docker {
      image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
      reuseNode true
    }
  }
  steps {
    sh 'npm install'
    sh 'node_modules/.bin/serve -s build &'
    sh 'sleep 10'
    sh 'npx playwright test --reporter=html'
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
             npm install netlify-cli -g
             netlify --version
              '''
            }
        }
    }
    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
}