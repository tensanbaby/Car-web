pipeline {
  agent any
  
  parameters {
    string(name: 'DOCKER_IMAGE_NAME', defaultValue: 'car-image', description: 'Name of the image')
  }
  
  stages {
    stage('Checkout Application Code') {
      steps {
        script {
          checkout([$class: 'GitSCM', branches: [[name: '*/*']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'jenkins-git', url: 'https://github.com/arunawsdevops/Car-web.git']]])
        }
      }
    }

    stage('Build docker image') {
      steps {
        script {
          sh "docker build -t ${params.DOCKER_IMAGE_NAME}:latest . -f Dockerfile"
        }
      }
    }

    stage('Push to Dockerhub') {
      steps {
        script {
          withCredentials([usernamePassword(credentialsId: 'dockerhub-devops', passwordVariable: 'DOCKER_HUB_PASSWORD', usernameVariable: 'DOCKER_HUB_USERNAME')]) {
            sh "docker login -u ${DOCKER_HUB_USERNAME} -p '${DOCKER_HUB_PASSWORD}'"
            
            sh "docker tag ${params.DOCKER_IMAGE_NAME}:latest ${env.DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
            sh "docker push ${DOCKER_HUB_USERNAME}/${params.DOCKER_IMAGE_NAME}:latest"
          }
        }
      }
    }
  }
}
