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
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'jenkins-git', url: 'https://github.com/arunawsdevops/Car-web.git']]])
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
                    // Assuming 'dockerhub-devops' is the ID of your Docker Hub credentials
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-devops', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
                        // Log in to Docker Hub using provided credentials
                        sh "echo '${DOCKER_HUB_PASSWORD}' | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                        
                        // Tag and push the built Docker image to Docker Hub
                        sh "docker tag ${params.DOCKER_IMAGE_NAME}:latest ${DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
                        sh "docker push ${DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
                    }
                }
            }
        }
    }
}
