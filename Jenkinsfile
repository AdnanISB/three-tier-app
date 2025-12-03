@Library('sharedLib') _
pipeline {
    agent any

    environment {
        IMAGE_TAG   = "${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
        COMPOSE_FILE = "docker-compose.yml"
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkoutSource()
            }
        }

        stage('Docker Hub Login') {
            steps {
                 dockerLogin('dockerhub-credentials')
                }
            }

        stage('Build & Tag Images') {
            steps {
              buildImages('./backend', './frontend', IMAGE_TAG)
            }
        }

        stage('Push Images to Docker Hub') {
            steps {
                pushImages()
            }
        }

        stage('Prepare .env for Compose') {
            steps {
               prepareEnvFile()
            }
        }
        
        stage('Deploy Environment') {
            steps {
               script{
                   deployApp(COMPOSE_FILE, env.BRANCH_NAME)
               }
            }
        }

        stage('Cleanup Local Images') {
            steps {
               cleanupImages()
            }
        }
    }

    post {
        success {
            echo "✅ ${BRANCH_NAME} environment deployed successfully using Docker Hub images!"
        }
        failure {
            echo "❌ Deployment failed for ${BRANCH_NAME}. Check logs."
        }
    }
}
