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
  }
}
