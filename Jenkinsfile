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
                    // This line finds "IMAGE_TAG" in your file and replaces it with the actual Jenkins Build Number
                    sh "sed -i 's|IMAGE_TAG|${BUILD_NUMBER}|g' deployment.yaml"
                    
                    sh 'kubectl apply -f deployment.yaml --kubeconfig="$KUBECONFIG_PATH"'
                    
                    // You don't need 'rollout restart' if you use dynamic tags, 
                    // because K8s sees a new version number and updates automatically.
                }
            }
        }
    }
}
