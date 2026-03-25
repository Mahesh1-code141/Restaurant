pipeline {
    agent any

    environment {
        RECIPIENTS = 'maheshbabuya@gmail.com'
        GIT_REPO = 'https://github.com/Mahesh1-code141/Restaurant.git'
        GIT_BRANCH = 'main'
        KUBE_CONFIG = '/home/jenkins/.kube/config' // path to kubeconfig on Jenkins agent
        KUBE_NAMESPACE = 'mahesh'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out code from Git...'
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO}"
            }
        }

        stage('Build') {
            steps {
                echo 'Building the application...'
                // Example: if it's a Dockerized app
                sh 'docker build -t myapp:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                echo 'Pushing Docker image to registry...'
                // Replace with your registry details
                sh 'docker tag myapp:${BUILD_NUMBER} myregistry.com/myapp:${BUILD_NUMBER}'
                sh 'docker push myregistry.com/myapp:${BUILD_NUMBER}'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes...'
                withKubeConfig([credentialsId: 'kube-credentials', serverUrl: '', namespace: "${KUBE_NAMESPACE}"]) {
                    sh """
                    kubectl set image deployment/myapp-deployment myapp=myregistry.com/myapp:${BUILD_NUMBER} -n ${KUBE_NAMESPACE}
                    kubectl rollout status deployment/myapp-deployment -n ${KUBE_NAMESPACE}
                    """
                }
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
