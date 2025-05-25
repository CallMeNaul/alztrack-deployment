pipeline {
    agent any
    
    environment {
        DEPLOY_SCM = 'https://github.com/CallMeNaul/alztrack-deployment.git'
    }
    
    stages {
        stage('Info') {
            steps {
                sh(label: "ℹ️ Showing system info", script: """
                    whoami
                    pwd
                    ls
                """)
            }
        }

        stage('Checkout') {
            steps {
                cleanWs()
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: env.DEPLOY_SCM,
                        credentialsId: 'login-github'
                    ]]
                ])
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    sh(label: "🔄 Stopping old containers", script: "docker-compose down -v || true")
                    sh(label: "🚀 Starting new containers", script: "docker-compose up -d")
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    // Wait for services to be up
                    sh(label: "⏳ Waiting for services to start", script: "sleep 30")
                    
                    // Check if services are running
                    sh(label: "🔍 Checking Postgres container", script: "docker ps | grep alzheimer-postgres")
                    sh(label: "🔍 Checking Backend container", script: "docker ps | grep alzheimer-backend")
                    sh(label: "🔍 Checking Frontend container", script: "docker ps | grep alzheimer-app")
                    sh(label: "🔍 Checking Diagnosing Server container", script: "docker ps | grep alzheimer-diagnosing-server")
                    
                    // Basic health check for diagnosing server
                    sh(label: "🏥 Checking Diagnosing Server health", script: "curl -f http://localhost:8000/predict || true")
                }
            }
        }
    }
    
    post {
        success {
            echo '✅ Pipeline completed successfully!'
        }
        failure {
            // Roll back if deployment fails
            sh(label: "🔄 Rolling back deployment", script: "docker-compose down -v || true")
            echo '❌ Pipeline failed!'
        }
    }
} 
