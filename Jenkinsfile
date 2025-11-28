pipeline {
    agent any
    
    environment {
        BACKEND_IMAGE = 'your-username/backend-app'
        FRONTEND_IMAGE = 'your-username/frontend-app'
        DOCKER_REGISTRY = 'your-dockerhub-username'
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-username/your-fullstack-app.git'
            }
        }
        
        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                    script {
                        docker.build("${BACKEND_IMAGE}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Test Backend') {
            steps {
                dir('backend') {
                    // Add your backend tests here
                    sh 'echo "Backend tests would run here"'
                    // Example: sh 'npm test' if you had tests
                }
            }
        }
        
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                    script {
                        docker.build("${FRONTEND_IMAGE}:${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        
        stage('Test Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm test -- --watch=false --browsers=ChromeHeadless'
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                script {
                    sh "docker scan ${BACKEND_IMAGE}:${env.BUILD_NUMBER}"
                    sh "docker scan ${FRONTEND_IMAGE}:${env.BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                script {
                    // Stop and remove old containers
                    sh 'docker stop backend-app || true'
                    sh 'docker rm backend-app || true'
                    sh 'docker stop frontend-app || true'
                    sh 'docker rm frontend-app || true'
                    
                    // Create network if not exists
                    sh 'docker network create app-network || true'
                    
                    // Run backend
                    sh """
                    docker run -d \
                      --name backend-app \
                      --network app-network \
                      -p 8080:8080 \
                      ${BACKEND_IMAGE}:${env.BUILD_NUMBER}
                    """
                    
                    // Run frontend
                    sh """
                    docker run -d \
                      --name frontend-app \
                      --network app-network \
                      -p 80:80 \
                      ${FRONTEND_IMAGE}:${env.BUILD_NUMBER}
                    """
                }
            }
        }
    }
    
    post {
        always {
            // Clean up
            sh 'docker system prune -f'
            
            // Publish test results
            junit 'frontend/test-results/**/*.xml'
            publishHTML([
                allowMissing: false,
                alwaysLinkToLastBuild: false,
                keepAll: true,
                reportDir: 'frontend/coverage',
                reportFiles: 'index.html',
                reportName: 'Frontend Coverage Report'
            ])
        }
        success {
            echo 'Full-stack application deployed successfully!'
            // Add notifications
            emailext (
                subject: "SUCCESS: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The full-stack application has been deployed successfully.\nCheck it at: http://${env.SERVER_IP}",
                to: "team@yourcompany.com"
            )
        }
        failure {
            echo 'Pipeline failed!'
            emailext (
                subject: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
                body: "The pipeline failed. Please check Jenkins for details.",
                to: "team@yourcompany.com"
            )
        }
    }
}