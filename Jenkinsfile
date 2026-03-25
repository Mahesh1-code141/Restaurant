pipeline {
    agent any

    environment {
        RECIPIENTS          = 'maheshbabuyarramsetti09@gmail.com'
        GIT_REPO            = 'https://github.com/Mahesh1-code141/Restaurant.git'
        GIT_BRANCH          = 'main'
        KUBE_NAMESPACE      = 'mahesh'
        DOCKER_REGISTRY     = 'docker.io/mahesh2452/restaurant'  // full image path
        DOCKER_LOGIN_SERVER = 'docker.io'
        DOCKER_IMAGE        = 'restaurant'
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
                sh "docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} ."
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

                        docker tag ${DOCKER_IMAGE}:${BUILD_NUMBER} \
                            ${DOCKER_REGISTRY}:${BUILD_NUMBER}

                        docker push ${DOCKER_REGISTRY}:${BUILD_NUMBER}
                    """
                }
            }
        }

        stage('Pre-Deployment Notification') {
            steps {
                emailext (
                    subject: "Deployment Starting: ${DOCKER_IMAGE}:${BUILD_NUMBER}",
                    body:    "Deployment of ${DOCKER_IMAGE}:${BUILD_NUMBER} to Kubernetes namespace '${KUBE_NAMESPACE}' is starting.",
                    to:      "${RECIPIENTS}"
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
  name: ${DOCKER_IMAGE}-deployment
  namespace: ${KUBE_NAMESPACE}
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ${DOCKER_IMAGE}
  template:
    metadata:
      labels:
        app: ${DOCKER_IMAGE}
    spec:
      containers:
      - name: ${DOCKER_IMAGE}
        image: ${DOCKER_REGISTRY}:${BUILD_NUMBER}
        imagePullPolicy: Always
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: ${DOCKER_IMAGE}-service
  namespace: ${KUBE_NAMESPACE}
spec:
  type: NodePort
  selector:
    app: ${DOCKER_IMAGE}
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
EOF
            """

            sh "kubectl rollout status deployment/${DOCKER_IMAGE}-deployment -n ${KUBE_NAMESPACE}"
        }
    }
}
