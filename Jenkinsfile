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
                        sh "docker tag ${BACKEND_IMG}:${BUILD_NUMBER} ${BACKEND_IMG}:latest"
                        sh "docker push ${BACKEND_IMG}:${BUILD_NUMBER}"
                        sh "docker push ${BACKEND_IMG}:latest"
                    }
                    dir('frontend') {
                        sh "docker build -t ${FRONTEND_IMG}:${BUILD_NUMBER} ."
                        sh "docker tag ${FRONTEND_IMG}:${BUILD_NUMBER} ${FRONTEND_IMG}:latest"
                        sh "docker push ${FRONTEND_IMG}:${BUILD_NUMBER}"
                        sh "docker push ${FRONTEND_IMG}:latest"
                    }
                }
            }
        }
        
        stage('Deploy to AKS') {
            steps {
                withCredentials([file(credentialsId: 'aks-kubeconfig-file', variable: 'KUBECONFIG_PATH')]) {
                    // FIX: Added 'k8s/' folder prefix. Adjust if your YAMLs are in root.
                    sh 'kubectl apply -f deployment.yaml --kubeconfig="$KUBECONFIG_PATH"'
                    
                    // Force restart to pull the 'latest' image
                    sh 'kubectl rollout restart deployment backend --kubeconfig="$KUBECONFIG_PATH"'
                    sh 'kubectl rollout restart deployment frontend --kubeconfig="$KUBECONFIG_PATH"'
                }
            }
        }
    }
}
