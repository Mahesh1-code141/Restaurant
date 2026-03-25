pipeline {
    agent any

    environment {
        RECIPIENTS = 'maheshbabuya@gmail.com'
        GIT_REPO = 'https://github.com/Mahesh1-code141/Restaurant.git'
        GIT_BRANCH = 'main'
        KUBE_NAMESPACE = 'mahesh'
        DOCKER_REGISTRY = 'mahesh2452'
        DOCKER_IMAGE = 'restaurant'
        DOCKER_CREDENTIALS_ID = 'Docker_CRED'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git...'
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .'
            }
        }

        stage('Login & Push Docker Image') {
            steps {
                echo 'Logging into Docker registry and pushing image...'
                withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login ${DOCKER_REGISTRY} -u "$DOCKER_USER" --password-stdin
                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${BUILD_NUMBER}
                    '''
                }
            }
        }

        stage('Pre-Deployment Notification') {
            steps {
                emailext (
                    subject: "Deployment Starting: ${DOCKER_IMAGE}:${BUILD_NUMBER}",
                    body: "Deployment of ${DOCKER_IMAGE}:${BUILD_NUMBER} to Kubernetes namespace ${KUBE_NAMESPACE} is starting.",
                    to: "${RECIPIENTS}"
                )
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                script {
                    def rolloutStatus = sh(
                        script: """
                            kubectl set image deployment/${DOCKER_IMAGE}-deployment ${DOCKER_IMAGE}=${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${BUILD_NUMBER} -n ${KUBE_NAMESPACE}
                            kubectl rollout status deployment/${DOCKER_IMAGE}-deployment -n ${KUBE_NAMESPACE}
                        """,
                        returnStdout: true
                    ).trim()

                    // Post-deployment email
                    emailext (
                        subject: "Deployment Complete: ${DOCKER_IMAGE}:${BUILD_NUMBER}",
                        body: "Deployment finished successfully.\n\nRollout Status:\n${rolloutStatus}",
                        to: "${RECIPIENTS}"
                    )
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully."
        }
        failure {
            emailext (
                subject: "Jenkins Job '${env.JOB_NAME}' Failed",
                body: "Alert! Job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) failed.\n\nCheck console output at ${env.BUILD_URL}",
                to: "${RECIPIENTS}"
            )
        }
        unstable {
            emailext (
                subject: "Jenkins Job '${env.JOB_NAME}' Unstable",
                body: "Job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) is unstable.\n\nCheck console output at ${env.BUILD_URL}",
                to: "${RECIPIENTS}"
            )
        }
    }
}
