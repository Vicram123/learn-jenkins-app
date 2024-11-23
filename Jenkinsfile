pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v $PWD:/workspace -w /workspace'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Listing files in workspace:"
                ls -la
                echo "Node version:"
                node --version
                echo "NPM version:"
                npm --version

                echo "Installing dependencies..."
                npm ci || { echo "Dependency installation failed"; exit 1; }

                echo "Building project..."
                npm run build || { echo "Build failed"; exit 1; }

                echo "Listing build files:"
                ls -la build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v $PWD:/workspace -w /workspace'
                    reuseNode true
                }
            }
            steps {
                sh '''
                echo "Checking for build output..."
                test -f build/index.html || { echo "Build output missing"; exit 1; }

                echo "Running tests..."
                npm test || { echo "Tests failed"; exit 1; }
                '''
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:18-alpine'
                    args '-v $PWD:/workspace -w /workspace'
                    reuseNode true
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('NETLIFY_AUTH_TOKEN')
                NETLIFY_SITE_ID = 'fd05daaa-65c0-40c3-9727-3505b0757b79'
            }
            steps {
                sh '''
                        # Install Netlify CLI locally
                        npm install netlify-cli || { echo "Failed to install Netlify CLI"; exit 1; }

                        # Check Netlify CLI version
                        node_modules/.bin/netlify --version

                        # Notify about the deployment
                        echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"

                        # Perform the deployment
                        echo "Deploying to Netlify..."
                        node_modules/.bin/netlify deploy --auth $NETLIFY_AUTH_TOKEN --site $NETLIFY_SITE_ID --prod --dir=build || { echo "Deployment failed"; exit 1; }

                        # Check the status
                        node_modules/.bin/netlify status

                        # Notify successful deployment
                        echo "Deployment successful."
                    '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/build/**', fingerprint: true
            sh 'test -f test-results/junit.xml && echo "JUnit results found" || echo "No JUnit results"'
            junit allowEmptyResults: true, testResults: 'test-results/junit.xml'
        }
    }
}
