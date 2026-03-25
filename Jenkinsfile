pipeline {
    agent any

    environment {
        RECIPIENTS = 'maheshbabuya@gmail.com'
        GIT_REPO = 'https://github.com/Mahesh1-code141/Restaurant.git'
        GIT_BRANCH = 'main'
        KUBE_NAMESPACE = 'mahesh'
        DOCKER_REGISTRY = 'myregistry.com'
        DOCKER_IMAGE = 'restaurant'
        DOCKER_CREDENTIALS_ID = 'Docker_CRED' // Jenkins stored credentials
    }

    triggers {
        // Trigger on GitHub push events
        githubPush()
        // Optional: poll every 5 minutes if webhook not available
        // pollSCM('H/5 * * * *')
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

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                sh """
                    kubectl set image deployment/${DOCKER_IMAGE}-deployment ${DOCKER_IMAGE}=${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${BUILD_NUMBER} -n ${KUBE_NAMESPACE}
                    kubectl rollout status deployment/${DOCKER_IMAGE}-deployment -n ${KUBE_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            emailext (
                subject: "Jenkins Job '${env.JOB_NAME}' Success",
                body: "Good news! Job '${env.JOB_NAME}' (#${env.BUILD_NUMBER}) succeeded.\n\nCheck console output at ${env.BUILD_URL}",
                to: "${RECIPIENTS}"
            )
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
