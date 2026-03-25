pipeline {
    agent any

    environment {
        RECIPIENTS = 'rajeshtutta7176@gmail.com'
    }

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                // your build steps here
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                // your test steps here
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
