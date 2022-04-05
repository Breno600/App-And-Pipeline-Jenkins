pipeline {
  environment {
    registry = "breno600/labs-frontend"
    registryCredential = 'docker-login'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        git 'https://github.com/Breno600/labs-breno.git'
      }
    }
    stage('Building image') {
      steps{
        script {
          docker ps
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Deploy Image') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
        sh "curl https://127.0.0.1:6443"
        sh "ls"
      }
    }
    
    stage('Approve Deploy to HML?') {
      when {
        beforeAgent true
        anyOf {
          branch "master"
        }
      }
      options {
        timeout(time: 7, unit: 'DAYS')
      }
      steps{
        script {
          timeout(time: 7, unit: 'DAYS') {
            userInput = input(
              id: 'Proceed1', message: 'Was this successful?', parameters: [
                [$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Confirm deploy?']
              ])
          }
        }
      }
      post {
        success {
          echo "Approved by "
        }
        aborted {
          echo "Declined by "
        }
      }
    }
    
    stage('Deploy HML K3S') {
      when{
        beforeAgent true
        anyOf {
          branch "master"
        }
        expression { return ( userInput == true ) }
      }
      steps{
        sh '''
          apt-get update
          apt-get upgrade -y
          apt get install sshpass -y
          sshpass -p 'breno123' ssh pro@192.168.0.14 './script_deploy.sh'
          '''
      }
    }
    
  }
}
