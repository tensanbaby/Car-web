pipeline {
    agent any

    parameters {
        string(name: 'AWS_REGION', defaultValue: 'us-east-1', description: 'AWS Region')
        string(name: 'AWS_ACCOUNT_ID', defaultValue: 'your-aws-account-id', description: 'AWS Account ID')
    }

    environment {
        AWS_DEFAULT_REGION = "${params.AWS_REGION}"
    }

    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'GIT-PAT-1', url: 'https://github.com/arunawsdevops/Project-Infra-autodesk.git'
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
                    dockerImage = docker.build("your-ecr-repo-name:${params.AWS_REGION}")
                }
            }
        }
        stage('Push to ECR') {
            steps {
                 script {
                     sh """
                         \$(aws ecr get-login-password --region \${params.AWS_REGION} | docker login --username AWS --password-stdin \${params.AWS_ACCOUNT_ID}.dkr.ecr.\${params.AWS_REGION}.amazonaws.com)
                         docker push your-ecr-repo-name:${params.AWS_REGION}
                        """
                }
            }
        }
        stage('Update ECS Service') {
            steps {
                script {
                    sh """
                    aws ecs update-service --cluster green-app-cluster --service green-app-service --force-new-deployment
                    """
                }
            }
        }
    }
}
