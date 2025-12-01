pipeline {
    agent { label 'ec2-dev' }

    environment {
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
        BRANCH_NAME = "${env.BRANCH_NAME}"
        IMAGE_TAG   = "${BRANCH_NAME}-${BUILD_NUMBER}"
        DOCKERHUB_USER = 'addd275'
    }

    options {
        skipStagesAfterUnstable()
        timestamps()
    }

    stages {

        stage('Pipeline with Color') {
            steps {
                wrap([$class: 'AnsiColorBuildWrapper', 'colorMapName': 'xterm']) {

                    // ----- REAL PIPELINE STARTS HERE -----

                    script {
                        echo "🔵 Colors are now working!"
                    }

                    checkout scm

                    withCredentials([usernamePassword(
                        credentialsId: "${DOCKERHUB_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_PASS'
                    )]) {
                        sh '''
                            echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                        '''
                        script { env.DOCKERHUB_USER = "${DOCKERHUB_USER}" }
                    }

                    script {
                        env.BACKEND_TAG_DH  = "${DOCKERHUB_USER}/three-tier-app-backend:${IMAGE_TAG}"
                        env.FRONTEND_TAG_DH = "${DOCKERHUB_USER}/three-tier-app-frontend:${IMAGE_TAG}"
                    }

                    sh '''
                        docker build -t ${BACKEND_TAG_DH} ./backend
                        docker build -t ${FRONTEND_TAG_DH} ./frontend
                    '''

                    sh '''
                        docker push ${BACKEND_TAG_DH}
                        docker push ${FRONTEND_TAG_DH}
                    '''

                    writeFile(
                        file: '.env',
                        text: """
                        BACKEND_IMAGE=${BACKEND_TAG_DH}
                        FRONTEND_IMAGE=${FRONTEND_TAG_DH}
                        """
                    )

                    sh """
                        docker-compose -f docker-compose.yml --env-file .env down
                        docker-compose -f docker-compose.yml --env-file .env pull
                        docker-compose -f docker-compose.yml --env-file .env up -d --remove-orphans
                    """

                    sh 'docker rmi ${BACKEND_TAG_DH} ${FRONTEND_TAG_DH} || true'

                    // ----- REAL PIPELINE ENDS HERE -----

                }
            }
        }
    }

    post {
        success { echo "✅ ${BRANCH_NAME} deployed successfully! Tag: ${IMAGE_TAG}" }
        failure { echo "❌ Deployment failed. Check logs." }
    }
}
