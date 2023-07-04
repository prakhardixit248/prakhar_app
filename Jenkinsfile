pipeline {
  agent {
    label 'worker'
  }
   
  stages {
    stage('Git Checkout') {
      steps {
        checkout([$class: 'GitSCM',
                  branches: [[name: '*/v1']],
                  userRemoteConfigs: [[url: 'https://github.com/prakhardixit248/prakhar_app']]])
      }
    }
    
     stage('Stopping the container') {
      steps {
        script {
          sh 'docker kill $(docker ps -q)'
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {     
          sh '''
		  aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 166422601531.dkr.ecr.us-east-1.amazonaws.com
		  docker build -t bastionproject:${BUILD_NUMBER} .
		  docker tag bastionproject:${BUILD_NUMBER} 166422601531.dkr.ecr.us-east-1.amazonaws.com/bastionproject:v${BUILD_NUMBER}
          docker push 166422601531.dkr.ecr.us-east-1.amazonaws.com/bastionproject:v${BUILD_NUMBER}
          '''
        }
      }
    }

    stage('Cleanup the docker image') {
      steps {
        script {
          sh 'docker rmi 166422601531.dkr.ecr.us-east-1.amazonaws.com/bastionproject:v${BUILD_NUMBER}:${BUILD_NUMBER}'
          sh 'docker rmi bastionproject:${BUILD_NUMBER}'
        }
      }
    }

    stage('Deploy the application') {
      steps {
        script {
          sh '''
          aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 166422601531.dkr.ecr.us-east-1.amazonaws.com/bastionproject:v${BUILD_NUMBER}
          docker pull 166422601531.dkr.ecr.us-east-1.amazonaws.com/bastionproject:v${BUILD_NUMBER}
          docker run -d -p 8080:8081 166422601531.dkr.ecr.us-east-1.amazonaws.com/bastionproject:v${BUILD_NUMBER}
          '''
        }
      }
    }
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timeout(time: 1, unit: 'HOURS')
  }
}
