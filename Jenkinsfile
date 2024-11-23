pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:19'
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
                    image 'node:19'
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
                    image 'node:19'
                    args '-v $PWD:/workspace -w /workspace'
                    reuseNode true
                }
            }
            environment {
                NETLIFY_AUTH_TOKEN = credentials('nfp_GsbbHHGhFMmTJAe7JTVEZ7ddKUrzMpTz4e51')
                SITE_ID = credentials('fd05daaa-65c0-40c3-9727-3505b0757b79')
            }
            steps {
                sh '''
                echo "Ensuring build directory exists..."
                test -d build || { echo "Build directory missing"; exit 1; }

                echo "Installing Netlify CLI..."
                npm install -g netlify-cli || { echo "Failed to install Netlify CLI"; exit 1; }

                echo "Deploying to Netlify..."
                netlify deploy --auth $NETLIFY_AUTH_TOKEN --site $SITE_ID --prod --dir=build || { echo "Deployment failed"; exit 1; }

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
