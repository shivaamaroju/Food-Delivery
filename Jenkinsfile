pipeline {
    agent any
    environment {
        DOCKER_USER = "shivaamaroju" 
        BACKEND_IMG = "${DOCKER_USER}/food-backend"
        FRONTEND_IMG = "${DOCKER_USER}/food-frontend"
    }
    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-token', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                }
            }
        }
        
        stage('Build & Push Images') {
            steps {
                script {
                    dir('backend') {
                        sh "docker build -t ${BACKEND_IMG}:${BUILD_NUMBER} ."
                        sh "docker push ${BACKEND_IMG}:${BUILD_NUMBER}"
                        // Also push latest for redundancy
                        sh "docker tag ${BACKEND_IMG}:${BUILD_NUMBER} ${BACKEND_IMG}:latest"
                        sh "docker push ${BACKEND_IMG}:latest"
                    }
                    dir('frontend') {
                        sh "docker build -t ${FRONTEND_IMG}:${BUILD_NUMBER} ."
                        sh "docker push ${FRONTEND_IMG}:${BUILD_NUMBER}"
                        sh "docker tag ${FRONTEND_IMG}:${BUILD_NUMBER} ${FRONTEND_IMG}:latest"
                        sh "docker push ${FRONTEND_IMG}:latest"
                    }
                }
            }
        }
        
        stage('Deploy to AKS') {
            steps {
                withCredentials([
                    file(credentialsId: 'aks-kubeconfig-file', variable: 'KUBECONFIG_PATH'),
                    string(credentialsId: 'mongo-url-secret', variable: 'MONGO_URI')
                ]) {
                    script {
                        // 1. Create or Update ConfigMap (No file needed in GitHub)
                        sh """
                        kubectl create configmap backend-config \
                          --from-literal=MONGO_URL='${MONGO_URI}' \
                          --dry-run=client -o yaml | kubectl apply -f - --kubeconfig='${KUBECONFIG_PATH}'
                        """

                        // 2. Replace IMAGE_TAG placeholder with actual Build Number
                        sh "sed -i 's|IMAGE_TAG|${BUILD_NUMBER}|g' deployment.yaml"
                        
                        // 3. Apply Deployment
                        sh "kubectl apply -f deployment.yaml --kubeconfig='${KUBECONFIG_PATH}'"
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo "Successfully deployed build #${BUILD_NUMBER} to AKS!"
        }
    }
}
