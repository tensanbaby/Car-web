pipeline {
    agent any

    parameters {
        choice(name: 'TERRAFORM_ACTION', choices: ['apply', 'destroy'], description: 'Select Terraform action to perform')
        string(name: 'USER_NAME', defaultValue: 'Arun', description: 'Specify who is running the code')
    }

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        AWS_DEFAULT_REGION    = 'us-east-1'  // Set AWS region
        TF_VAR_aws_region     = 'us-east-1'  // Set Terraform variable
    }

    stages {
        stage('Checkout Terraform Code') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'github-pat', variable: 'GITHUB_PAT')]) {
                        sh "git clone https://$GITHUB_PAT@github.com/arunawsdevops/Car-web.git"
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                dir('Project-Infra-autodesk') {
                    withCredentials([string(credentialsId: 'aws-access-key-id', variable: 'AWS_ACCESS_KEY_ID'), 
                                     string(credentialsId: 'aws-secret-access-key', variable: 'AWS_SECRET_ACCESS_KEY')]) {
                        sh 'terraform init'
                    }
                }
            }
        }

        stage('Terraform Action') {
            steps {
                script {
                    if (params.TERRAFORM_ACTION == 'apply') {
                        dir('Project-Infra-autodesk') {
                            sh 'terraform apply -auto-approve'
                        }
                    } else if (params.TERRAFORM_ACTION == 'destroy') {
                        dir('Project-Infra-autodesk') {
                            sh 'terraform destroy -auto-approve'
                        }
                    } else {
                        error "Invalid Terraform action selected: ${params.TERRAFORM_ACTION}"
                    }
                }
            }
        }

        stage('Checkout Application Code') {
            when {
                expression {
                    return params.TERRAFORM_ACTION == 'apply'
                }
            }
            steps {
                script {
                    // Retrieve dynamic values from AWS
                    def AWS_ACCOUNT_ID = sh(script: "aws sts get-caller-identity --query Account --output text", returnStdout: true).trim()
                    def ECR_REPO_NAME = 'test-project-repo' // Replace with dynamic retrieval if possible
                    def ECS_CLUSTER_NAME = 'green-app-cluster' // Replace with dynamic retrieval if possible
                    def ECS_SERVICE_NAME = 'green-app-service' // Replace with dynamic retrieval if possible
                    
                    echo "AWS Account ID: ${AWS_ACCOUNT_ID}"
                    echo "ECR Repo Name: ${ECR_REPO_NAME}"
                    echo "ECS Cluster Name: ${ECS_CLUSTER_NAME}"
                    echo "ECS Service Name: ${ECS_SERVICE_NAME}"
                    
                    // Checkout application code
                    checkout([$class: 'GitSCM', branches: [[name: '*/*']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'jenkins-git', url: 'https://github.com/arunawsdevops/Car-web.git']]])

                }
            }
        }
        
        stage('Build and Push to ECR') {
            when {
                expression {
                    return params.TERRAFORM_ACTION == 'apply'
                }
            }
            steps {
                script {
                    withAWS(credentials: 'aws-cred', region: AWS_DEFAULT_REGION) {
                        def ecrLogin = sh(script: "aws ecr get-login-password --region ${AWS_DEFAULT_REGION}", returnStdout: true).trim()
                        sh "docker login -u AWS -p '${ecrLogin}' ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                        sh "docker build -t ${ECR_REPO_NAME} . -f Dockerfile"
                        sh "docker tag ${ECR_REPO_NAME}:latest ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:latest"
                        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPO_NAME}:latest"
                    }
                }
            }
        }

        stage('Update ECS Task Definition') {
            when {
                expression {
                    return params.TERRAFORM_ACTION == 'apply'
                }
            }
            steps {
                script {
                    withAWS(credentials: 'aws-cred', region: AWS_DEFAULT_REGION) {
                        // Update ECS task definition to use the newly pushed Docker image
                        sh "aws ecs register-task-definition --cli-input-json file://ecs-task-definition.json --region ${AWS_DEFAULT_REGION}"
                    }
                }
            }
        }

        stage('Deploy to ECS') {
            when {
                expression {
                    return params.TERRAFORM_ACTION == 'apply'
                }
            }
            steps {
                script {
                    withAWS(credentials: 'aws-cred', region: AWS_DEFAULT_REGION) {
                        sh "aws ecs update-service --cluster ${ECS_CLUSTER_NAME} --service ${ECS_SERVICE_NAME} --force-new-deployment --region ${AWS_DEFAULT_REGION}"
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
}
