# Orange Project 2 - Jenkins, Docker, and Kubernetes

## Overview
This project utilizes Jenkins to build Docker images from Dockerfiles in a GitHub repository, push these images to Docker Hub, and deploy them on a Kubernetes cluster using Minikube.

## Prerequisites
- Docker installed on the Jenkins Agent.
- A Kubernetes cluster (using Minikube in this case).
- Jenkins set up with proper access to the Kubernetes API.
- A GitHub account to host the project.
- A Docker Hub account for pushing images.

## Steps

### 1. Setting Up Jenkins in Kubernetes
1. Create a namespace for Jenkins in the Kubernetes cluster:
    ```bash
    kubectl create namespace jenkins
    ```

2. Create the `jenkins-deployment.yaml` and `jenkins-service.yaml` files to run Jenkins as a NodePort service:
    - **jenkins-deployment.yaml**:
      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: jenkins
        namespace: jenkins
      spec:
        replicas: 1
        selector:
          matchLabels:
            app: jenkins
        template:
          metadata:
            labels:
              app: jenkins
          spec:
            serviceAccountName: jenkins
            containers:
            - name: jenkins
              image: jenkins/jenkins:lts
              ports:
              - containerPort: 8080
              volumeMounts:
              - name: jenkins-volume
                mountPath: /var/jenkins_home
            volumes:
            - name: jenkins-volume
              emptyDir: {}
      ```

    - **jenkins-service.yaml**:
      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: jenkins-service
        namespace: jenkins
      spec:
        type: NodePort
        ports:
          - port: 8080
            targetPort: 8080
            nodePort: 30000
        selector:
          app: jenkins
      ```

3. Apply the YAML files:
    ```bash
    kubectl apply -f jenkins-deployment.yaml
    kubectl apply -f jenkins-service.yaml
    ```

### 2. Setting Up the Service Account and Permissions for Jenkins
1. Create a `jenkins-rbac.yaml` file to grant Jenkins the necessary permissions:
    ```yaml
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: jenkins
      namespace: jenkins
    ---
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: jenkins
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: cluster-admin
    subjects:
    - kind: ServiceAccount
      name: jenkins
      namespace: jenkins
    ```

2. Apply the RBAC configuration:
    ```bash
    kubectl apply -f jenkins-rbac.yaml
    ```

3. Create a token for the `jenkins` ServiceAccount:
    ```bash
    kubectl create token jenkins -n jenkins
    ```
   Save the token to be used in the Jenkins configuration.

### 3. Configuring Jenkins
1. Access the Jenkins dashboard using the Node IP and the specified port:
    ```
    http://<Node_IP>:30000
    ```

2. Install necessary plugins in Jenkins:
    - Kubernetes Plugin
    - Docker Pipeline Plugin
    - Git Plugin

3. Add Docker Hub and Kubernetes credentials in Jenkins:
    - Navigate to `Manage Jenkins > Manage Credentials`.
    - Add the Docker Hub credentials (Username and Password).
    - Add the Kubernetes token generated earlier as a "Secret Text."

4. Set up a Kubernetes cloud in Jenkins:
    - Go to `Manage Jenkins > Manage Nodes and Clouds > Configure Clouds > Add a new cloud > Kubernetes`.
    - Enter the Kubernetes cluster URL (e.g., `https://<MINIKUBE_IP>:8443`).
    - Select the namespace `jenkins` and the previously added Kubernetes token as credentials.

### 4. Creating a Jenkins Pipeline
1. Create a `Jenkinsfile` in the root of your GitHub repository to:
    - Checkout code from the repository.
    - Build Docker images for `backend` and `proxy` directly using the `docker build` command.
    - Push the images to Docker Hub.
    - Deploy the images to Kubernetes.

2. Example `Jenkinsfile`:
    ```groovy
    pipeline {
        agent any
        environment {
            GIT_REPO_URL = 'https://github.com/bebomm180/Orange-Project2.git'
            DOCKERHUB_CREDENTIALS = 'dockerhub-credentials-id' // Docker Hub credentials ID in Jenkins
            DOCKERHUB_USERNAME = 'bebomm180'
            K8S_NAMESPACE = 'default'
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
                        sh '''
                        echo "Building Backend Docker Image..."
                        docker build -t ${backendImage} ./backend
                        echo "Pushing Backend Docker Image to Docker Hub..."
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${backendImage}
                        '''
                    }
                }
            }
            stage('Build and Push Proxy Docker Image') {
                steps {
                    script {
                        def proxyImage = "${DOCKERHUB_USERNAME}/proxy:latest"
                        sh '''
                        echo "Building Proxy Docker Image..."
                        docker build -t ${proxyImage} ./proxy
                        echo "Pushing Proxy Docker Image to Docker Hub..."
                        echo "$DOCKERHUB_PASS" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push ${proxyImage}
                        '''
                    }
                }
            }
            stage('Deploy to Kubernetes') {
                steps {
                    script {
                        sh '''
                        echo "Applying Kubernetes configuration..."
                        kubectl apply -f k8s/deployment.yaml -n ${K8S_NAMESPACE}
                        kubectl apply -f k8s/service.yaml -n ${K8S_NAMESPACE}
                        ./apply-k8s.sh
                        '''
                    }
                }
            }
        }
        environment {
            DOCKERHUB_USER = credentials('dockerhub-credentials-id').username
            DOCKERHUB_PASS = credentials('dockerhub-credentials-id').password
        }
    }
    ```

### 5. Deploying on Kubernetes
- The pipeline applies the Kubernetes YAML files (`deployment.yaml` and `service.yaml`) to deploy the Docker images in the Kubernetes cluster.

## Conclusion
This project demonstrates how to use Jenkins to automate the build and deployment of Docker images from a GitHub repository to a Kubernetes cluster.



![image](https://github.com/user-attachments/assets/ce023b4e-b56b-459b-a397-aab70c389ead)

![image](https://github.com/user-attachments/assets/75092692-1499-4720-8933-2065396cc96f)


![image](https://github.com/user-attachments/assets/2f9b2720-82ac-442e-8ec0-cb0f45d42dbb)

![image](https://github.com/user-attachments/assets/0bc5c0da-804e-4a83-9e0f-dd5736b60e2c)

![image](https://github.com/user-attachments/assets/f2eabe3d-775c-43bc-abdf-34df3e8fdf50)










# thx

![image](https://github.com/user-attachments/assets/74cc6ab8-5bdc-4e5f-8b1c-dd54ed0d534c)

