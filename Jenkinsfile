pipeline {
    agent any

    environment {
        RECIPIENTS            = 'maheshbabuyarramsetti09@gmail.com'
        GIT_REPO              = 'https://github.com/Mahesh1-code141/Restaurant.git'
        GIT_BRANCH            = 'main'
        KUBE_NAMESPACE        = 'mahesh'
        DOCKER_IMAGE          = 'mahesh2452/restaurant'
        DOCKER_LOGIN_SERVER   = 'docker.io'
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
                sh """
                    docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} .
                    docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} ${DOCKER_IMAGE}:latest
                """
            }
        }

        stage('Login & Push Docker Image') {
            steps {
                echo 'Logging into Docker registry and pushing image...'
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login ${DOCKER_LOGIN_SERVER} \
                        -u "\$DOCKER_USER" --password-stdin

                        docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}
                        docker push ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }

        stage('Pre-Deployment Notification') {
            steps {
                emailext (
                    subject: "Deployment Starting: ${DOCKER_IMAGE}:${BUILD_NUMBER}",
                    body: "Deployment started for image ${DOCKER_IMAGE}:${BUILD_NUMBER} to namespace ${KUBE_NAMESPACE}.",
                    to: "${RECIPIENTS}"
                )
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo 'Deploying to Kubernetes...'

                    sh """
                        kubectl get namespace ${KUBE_NAMESPACE} || kubectl create namespace ${KUBE_NAMESPACE}

                        kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: restaurant-deployment
  namespace: ${KUBE_NAMESPACE}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: restaurant
  template:
    metadata:
      labels:
        app: restaurant
    spec:
      containers:
      - name: restaurant
        image: ${DOCKER_IMAGE}:${BUILD_NUMBER}
        imagePullPolicy: Always
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: restaurant-service
  namespace: ${KUBE_NAMESPACE}
spec:
  type: NodePort
  selector:
    app: restaurant
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
EOF
                    """

                    // Rollout with timeout
                    timeout(time: 2, unit: 'MINUTES') {
                        sh "kubectl rollout status deployment/restaurant-deployment -n ${KUBE_NAMESPACE}"
                    }

                    echo "Deployment successful!"
                }
            }
        }
    }

    post {
        success {
            emailext (
                subject: "✅ Deployment Successful: ${env.JOB_NAME}",
                body: "Deployment completed successfully!\n\nImage: ${DOCKER_IMAGE}:${BUILD_NUMBER}",
                to: "${RECIPIENTS}"
            )
        }

        failure {
            script {
                echo "Deployment failed! Rolling back..."

                sh """
                    kubectl rollout undo deployment/restaurant-deployment -n ${KUBE_NAMESPACE} || true
                """
            }

            emailext (
                subject: "❌ Deployment Failed: ${env.JOB_NAME}",
                body: "Deployment failed for build #${env.BUILD_NUMBER}.\nRollback attempted.\nCheck logs: ${env.BUILD_URL}",
                to: "${RECIPIENTS}"
            )
        }

        always {
            echo "Cleaning up Docker images..."
            sh "docker system prune -af || true"
        }
    }
}
