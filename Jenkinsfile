pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'car-image', description: 'Name of the Docker image')
    }

    environment {
        AWS_ACCESS_KEY_ID      = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY  = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION     = 'us-east-1'  // Set AWS region
    }

    stages {
        stage('Checkout Application Code') {
            steps {
                script {
                    // Checkout application code from the specified Git repository
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'jenkins-git', url: 'https://github.com/tensanbaby/Car-web.git']]])
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh "docker build -t ${params.DOCKER_IMAGE_NAME}:latest . -f Dockerfile"
                }
            }
        }

        stage('Push to Docker Hub') {
    steps {
        script {
            // Use Docker credentials to log in, tag, and push the Docker image
            withCredentials([usernamePassword(credentialsId: 'Docker_Pat', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                
                // Log in to Docker Hub using credentials stored in Jenkins
                sh "echo '${DOCKER_PASSWORD}' | docker login -u ${DOCKER_USERNAME} --password-stdin"
                
                // Tag the Docker image with the username and image name
                sh "docker tag ${params.DOCKER_IMAGE_NAME}:latest ${DOCKER_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
                
                // Push the Docker image to Docker Hub
                sh "docker push ${DOCKER_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
            }
        }
    }
}


       /* stage('Push to Docker Hub') {
            steps {
                script {
                    // Assuming 'dockerhub-devops' is the ID of your Docker Hub credentials
                    withCredentials([usernamePassword(credentialsId: 'Docker_Pat', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        // Log in to Docker Hub using provided credentials
                        sh "echo '${DOCKER_PASSWORD}' | docker login -u ${tensanbaby} --password-stdin"
                        
                        // Tag and push the built Docker image to Docker Hub
                        sh "docker tag ${params.DOCKER_IMAGE_NAME}:latest ${tensanbaby}/${params.DOCKER_IMAGE_NAME}:latest"
                       sh "docker push ${DOCKER_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest" 
                    }
                }
            }
        }*/

        stage('Deployment to EKS-cluster') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'EKS-CLUSTER', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl get nodes'
                        sh 'kubectl apply -f car-deploy.yaml'
                        sh 'kubectl apply -f car-svc.yaml'
                        sh 'kubectl get svc canary-svc -o wide'
                        sh 'kubectl scale deployment blue-app --replicas=3'

                    }
                }
            }
        }
    }
}
