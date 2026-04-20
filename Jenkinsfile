pipeline {
    agent any
    
    environment {
        DOCKER_USER = "shivaamaroju" // Change this
        FRONTEND_IMAGE = "food-frontend"
        BACKEND_IMAGE = "food-backend"
        AKS_CLUSTER = "myAKSCluster"
        RESOURCE_GROUP = "myResourceGroup"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/shivaamaroju/Food-Delivery.git'
            }
        }

        stage('Build & Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('', 'docker-hub-credentials') {
                        // Build and Push Backend
                        def backend = docker.build("${DOCKER_USER}/${BACKEND_IMAGE}:${BUILD_NUMBER}", "./backend")
                        backend.push()
                        
                        // Build and Push Frontend
                        def frontend = docker.build("${DOCKER_USER}/${FRONTEND_IMAGE}:${BUILD_NUMBER}", "./frontend")
                        frontend.push()
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    // Requires Azure CLI installed on Jenkins agent
                    sh "az aks get-credentials --resource-group ${RESOURCE_GROUP} --name ${AKS_CLUSTER}"
                    
                    // Replace placeholder in YAML and apply
                    sh "sed -i 's|IMAGE_TAG|${BUILD_NUMBER}|g' k8s/deployment.yaml"
                    sh "kubectl apply -f k8s/"
                }
            }
        }
    }
}