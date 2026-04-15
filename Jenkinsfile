pipeline {
    agent any
    
    tools {
        nodejs 'NodeJS-20'  // 🔧 CHANGED: Match your tool name
    }
    
    environment {
        APP_NAME = 'jenkins-demo-app'
        BUILD_NUMBER_ENV = "${BUILD_NUMBER}"
        DEPLOY_ENV = 'staging'
        // 🔥 ADDED: Dynamic port to prevent conflicts
        APP_PORT = "${3000 + BUILD_NUMBER.toInteger()}"
    }
    
    parameters {
        choice(
            name: 'ENVIRONMENT',
            choices: ['staging', 'production'],
            description: 'Select deployment environment'
        )
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip test execution'
        )
    }
    
    stages {
        stage('Pre-Cleanup') {
            steps {
                echo '🔥 Cleaning up previous processes...'
                // 🔧 CHANGED: Better kill command
                sh 'pkill -9 node || true'
                sh 'sleep 3'
            }
        }
        
        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub...'
                checkout scm
                sh 'ls -la'
                sh 'echo "Building for environment: ${ENVIRONMENT}"'
            }
        }
        
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'npm install'
                sh 'echo "Build completed for ${APP_NAME}"'
            }
        }
        
        stage('Test') {
            when {
                not { params.SKIP_TESTS }
            }
            steps {
                echo "Running tests on port ${APP_PORT}..."
                script {
                    // 🔧 CHANGED: Use dynamic port
                    sh "PORT=${APP_PORT} nohup node app.js > app.log 2>&1 &"
                    sh 'sleep 5'
                    // 🔧 CHANGED: Pass port to tests
                    sh "PORT=${APP_PORT} npm test"
                }
            }
            post {
                always {
                    echo 'Cleaning up test processes...'
                    sh 'pkill -9 node || true'
                    sh 'sleep 2'
                }
            }
        }
        
        stage('Deploy to Staging') {
            when {
                expression { params.ENVIRONMENT == 'staging' }
            }
            steps {
                echo 'Deploying to staging environment...'
                sh 'mkdir -p /tmp/staging-deployment'
                sh 'cp -r * /tmp/staging-deployment/ || true'
                sh 'echo "Deployed to staging at /tmp/staging-deployment"'
                sh 'ls -la /tmp/staging-deployment'
            }
        }
        
        stage('Deploy to Production') {
            when {
                expression { params.ENVIRONMENT == 'production' }
            }
            steps {
                echo 'Deploying to production environment...'
                sh 'mkdir -p /tmp/production-deployment'
                sh 'cp -r * /tmp/production-deployment/ || true'
                sh 'echo "Deployed to production at /tmp/production-deployment"'
                sh 'ls -la /tmp/production-deployment'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed!'
            sh 'echo "Build: ${BUILD_NUMBER_ENV}, Environment: ${ENVIRONMENT}"'
            sh 'pkill -9 node || true'
        }
        success {
            echo '✅ Pipeline executed successfully!'
        }
        failure {
            echo '❌ Pipeline execution failed!'
        }
    }
}
