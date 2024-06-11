pipeline {
    agent any

    parameters {
        string(name: 'AWS_REGION', defaultValue: 'us-east-1', description: 'AWS Region')
        string(name: 'ECS_CLUSTER_NAME', defaultValue: 'green-app-cluster', description: 'ECS Cluster Name')
        string(name: 'ECS_SERVICE_NAME', defaultValue: 'green-app-service', description: 'ECS Service Name')
        string(name: 'ECR_REPO_NAME', defaultValue: 'your-ecr-repo-name', description: 'ECR Repository Name')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag')
        string(name: 'AWS_ACCOUNT_ID', defaultValue: 'your-aws-account-id', description: 'AWS Account ID')
    }

    environment {
        AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION = "${params.AWS_REGION}"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/arunawsdevops/Project-Infra-autodesk.git'
            }
        }
        stage('Terraform Init and Apply') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'),
                                     string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh """
                        cd terraform-directory
                        terraform init
                        terraform apply -var 'aws_region=${params.AWS_REGION}' \
                                        -var 'ecs_cluster_name=${params.ECS_CLUSTER_NAME}' \
                                        -var 'ecr_repo_name=${params.ECR_REPO_NAME}' \
                                        -var 'image_tag=${params.IMAGE_TAG}' \
                                        -var 'aws_account_id=${params.AWS_ACCOUNT_ID}' \
                                        -auto-approve
                        """
                    }
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${params.ECR_REPO_NAME}:${params.IMAGE_TAG}")
                }
            }
        }
        stage('Push to ECR') {
            steps {
                 script {
                   sh """
                       \$(aws ecr get-login-password --region \${params.AWS_REGION} | docker login --username AWS --password-stdin \${params.AWS_ACCOUNT_ID}.dkr.ecr.\${params.AWS_REGION}.amazonaws.com)
                       docker push \${params.ECR_REPO_NAME}:\${params.IMAGE_TAG}
                      """
                }
            }
        }
        stage('Update ECS Service') {
            steps {
                script {
                    sh """
                    aws ecs update-service --cluster ${params.ECS_CLUSTER_NAME} --service ${params.ECS_SERVICE_NAME} --force-new-deployment
                    """
                }
            }
        }
    }
}
