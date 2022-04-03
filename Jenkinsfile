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
        sh "pwd"
        sh "ls"
      }
    }
    
    stage('Approve QA Deployment') {
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
    
    stage('Deploy homolog K3S') {
      when{
        beforeAgent true
        anyOf {
          branch "master"
        }
        expression { return ( userInput == true ) }
      }
      steps{
        sh "echo "Deu bom carai""
      }
    }
    
  }
}
