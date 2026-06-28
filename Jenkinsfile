node {
    def KUBECONFIG   = "$HOME/.kube/config"
    def REPO_URL     = "https://github.com/asabhaysingh328/myapp.git"
    def APP_NAME     = "myapp"
    def NAMESPACE    = "cicd-demo"
    def DOCKER_IMAGE = "asabhaysingh328/asabhaysingh328:latest"

    stage('Clone Repository') {
        git branch: 'main', url: REPO_URL
    }

    stage('Create Namespace') {
        sh "kubectl get ns ${NAMESPACE} || kubectl create namespace ${NAMESPACE}"
    }

    stage('Build Docker Image') {
        sh "docker build -t ${APP_NAME}:latest ."
    }

    stage('Push to Docker Hub') {
        withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
            sh """
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker tag ${APP_NAME}:latest ${DOCKER_IMAGE}
            docker push ${DOCKER_IMAGE}
            """
        }
    }

    stage('Deploy Application') {
        sh """
        kubectl get deploy ${APP_NAME} -n ${NAMESPACE} || \
        kubectl create deployment ${APP_NAME} --image=${DOCKER_IMAGE} -n ${NAMESPACE}
        kubectl expose deployment ${APP_NAME} --port=80 --target-port=80 -n ${NAMESPACE} || true
        """
    }

    stage('Verify Pods') {
        sh "kubectl get pods -n ${NAMESPACE} -o wide"
    }

    stage('Scale to 10 Pods') {
        sh "kubectl scale deployment ${APP_NAME} --replicas=10 -n ${NAMESPACE}"
    }

    stage('Rolling Restart') {
        sh "kubectl rollout restart deployment/${APP_NAME} -n ${NAMESPACE}"
    }

    stage('Rollout Status') {
        sh "kubectl rollout status deployment/${APP_NAME} -n ${NAMESPACE}"
    }

    stage('Check Deployment Health') {
        sh "kubectl get deploy ${APP_NAME} -n ${NAMESPACE} -o wide"
    }

    stage('Check Pod Logs') {
        sh "kubectl logs -n ${NAMESPACE} -l app=${APP_NAME} --tail=50"
    }

    stage('Final Verification') {
        sh "kubectl get pods -n ${NAMESPACE} -o wide"
    }
}
