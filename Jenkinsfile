pipeline {
    agent any

    parameters {
        string(name: 'AWS_REGION', defaultValue: 'us-east-1', description: 'AWS Region')
        string(name: 'AWS_ACCOUNT_ID', defaultValue: '6700-0448-7191', description: 'AWS Account ID')
        string(name: 'IMAGE_TAG', defaultValue: 'latest', description: 'Docker Image Tag')
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
    }
}
