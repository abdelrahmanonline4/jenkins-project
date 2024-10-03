pipeline {
    agent any
    environment {
        GIT_REPO_URL = 'https://github.com/abdelrahmanonline4/Orange-Project2.git'
        DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id' // استخدم الـ ID الخاص بالـ Docker Hub Credentials في Jenkins
        DOCKERHUB_USERNAME = 'bebomm180' // اسم المستخدم في Docker Hub
        K8S_NAMESPACE = 'webapp' // Namespace الذي ستستخدمه في Kubernetes
    }
    stages {
        stage('Checkout Code') {
            steps {
                git url: "${GIT_REPO_URL}"
            }
        }
        stage('Build and Push Backend Docker Image') {
            steps {
                script {
                    def backendImage = "${DOCKERHUB_USERNAME}/backend:latest"
                    sh """
                    echo "Building Backend Docker Image..."
                    docker build -t ${backendImage} ./backend
                    echo "Pushing Backend Docker Image to Docker Hub..."
                    echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                    docker push ${backendImage}
                    """
                }
            }
        }
        stage('Build and Push Proxy Docker Image') {
            steps {
                script {
                    def proxyImage = "${DOCKERHUB_USERNAME}/proxy:latest"
                    sh """
                    echo "Building Proxy Docker Image..."
                    docker build -t ${proxyImage} ./proxy
                    echo "Pushing Proxy Docker Image to Docker Hub..."
                    echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                    docker push ${proxyImage}
                    """
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                    echo "Applying Kubernetes configuration..."
                    kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}
                    kubectl apply -f k8s/service.yaml -n ${K8S_NAMESPACE}
                    ./apply-k8s.sh
                    """
                }
            }
        }
    }
    environment {
        DOCKERHUB_USER = credentials('dockerhub-credentials-id').username #add here 
        DOCKERHUB_PASS = credentials('dockerhub-credentials-id').password
    }
}

