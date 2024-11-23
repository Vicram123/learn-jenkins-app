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
        stage('Test') {
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
                    image 'node:19'
                    reuseNode true
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('nfp_GsbbHHGhFMmTJAe7JTVEZ7ddKUrzMpTz4e51')
                SITE_ID = credentials(' fd05daaa-65c0-40c3-9727-3505b0757b79')
            }
            steps {
                sh 'npm install -g netlify-cli'
                sh 'echo "Logging in to Netlify..."'
                sh 'netlify login --auth-token=$NETLIFY_AUTH_TOKEN'
                sh 'echo "Deploying to Netlify..."'
                sh 'netlify deploy --site $SITE_ID --auth $NETLIFY_AUTH_TOKEN --prod'
                sh 'echo "Deployment complete."'
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
