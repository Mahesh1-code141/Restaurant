stage('Deploy to Kubernetes') {
    steps {
        script {
            echo 'Deploying to Kubernetes...'

            sh """
                kubectl get deployment restaurant-deployment -n ${KUBE_NAMESPACE} || \
                kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: restaurant-deployment
  namespace: ${KUBE_NAMESPACE}
spec:
  replicas: 3
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
        image: ${DOCKER_REGISTRY}:${BUILD_NUMBER}
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: restaurant-service
  namespace: ${KUBE_NAMESPACE}
spec:
  type: LoadBalancer
  selector:
    app: restaurant
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF
            """

            // Update image if deployment exists
            sh """
                kubectl set image deployment/restaurant-deployment \
                restaurant=${DOCKER_REGISTRY}:${BUILD_NUMBER} \
                -n ${KUBE_NAMESPACE}
            """

            // Rollout status
            def rolloutStatus = sh(
                script: "kubectl rollout status deployment/restaurant-deployment -n ${KUBE_NAMESPACE}",
                returnStdout: true
            ).trim()

            echo "Rollout Status: ${rolloutStatus}"

            emailext (
                subject: "Deployment Complete: restaurant:${BUILD_NUMBER}",
                body: "Deployment finished successfully.\n\nRollout Status:\n${rolloutStatus}",
                to: "${RECIPIENTS}"
            )
        }
    }
}
