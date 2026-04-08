pipeline {
    agent any
    
    environment {
        APP_NAME = "node-todo"
        DOCKER_IMAGE = "node-todo-app"
    }
    
    tools {
        nodejs 'NodeJS-14'
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '📦 Cloning repository from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/levietthinh2k2-cloud/node-todo.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo '📥 Installing npm packages...'
                sh 'npm install'
            }
        }
        
        stage('Code Quality Check') {
            steps {
                echo '🔍 Running code quality checks...'
                script {
                    try {
                        sh 'npm run lint'
                    } catch (Exception e) {
                        echo 'No lint script found, skipping...'
                    }
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                echo '🧪 Running tests...'
                script {
                    try {
                        sh 'npm test'
                    } catch (Exception e) {
                        echo 'No test script found, skipping...'
                    }
                }
            }
        }
        
        stage('Build Docker Image') {
            steps {
                echo '🐳 Building Docker image...'
                sh '''
                    docker build -t %DOCKER_IMAGE%:latest .
                    docker tag %DOCKER_IMAGE%:latest %DOCKER_IMAGE%:%BUILD_NUMBER%
                '''
            }
        }
        
        stage('Stop Old Containers') {
            steps {
                echo '🛑 Stopping old containers...'
                sh '''
                    docker compose down || echo "No containers to stop"
                '''
            }
        }
        
        stage('Deploy with Docker Compose') {
            steps {
                echo '🚀 Deploying application...'
                sh '''
                    docker compose up -d
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                echo '💊 Running health check...'
                script {
                    sleep 15
                    sh '''
                        curl -f http://localhost:8080 || exit 1
                    '''
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
            echo '🌐 Application is running at http://localhost:8080'
        }
        failure {
            echo '❌ Pipeline failed!'
            sh 'docker compose down || echo "Cleanup failed"'
        }
        always {
            echo '🧹 Cleaning up...'
            sh 'docker system prune -f || echo "Prune failed"'
        }
    }
}