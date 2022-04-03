pipeline {
    //Global agent
    agent { node { label 'master' } }
    environment {
        IMAGE_REPOSITORY = 'breno600'
        IMAGE_NAME = 'labs-frontend'
        userInput = false
    }
    options {
        ansiColor('xterm')
        buildDiscarder(
            logRotator(
                // number of build logs to keep
                numToKeepStr:'5',
                // history to keep in days
                daysToKeepStr: '15',
                // artifacts are kept for days
                artifactDaysToKeepStr: '15',
                // number of builds have their artifacts kept
                artifactNumToKeepStr: '5'
            )
        )
    }

stages {

        stage('Build image Docker and push'){
            when {
                beforeAgent true
                anyOf {
                    branch "master"
                }
            }
            agent { node { label 'master' } }
            environment { IMAGE_TAG = GIT_COMMIT.take(7) }
            steps {
                script {
                    echo "${IMAGE_TAG}"
                    app = docker.build("${IMAGE_REPOSITORY}/${IMAGE_NAME}")
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-credentials') {
            		app.push("${IMAGE_TAG}")
        	     }
                }
            }
        }

        stage('Approve QA Deployment'){
            when {
                beforeAgent true
                anyOf {
                    branch "master"
                    branch "hotfix/*"
                }
            }
            options {

                timeout(time: 7, unit: 'DAYS')
            }
            steps {
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
        
        stage('Deploying to QA'){
            when{
                beforeAgent true
                anyOf {
                    branch "hotfix/*"
                }
                expression { return ( userInput == true ) }
            }
            agent { node { label 'master' } }
            environment { IMAGE_TAG = GIT_COMMIT.take(7) }
            steps {
                  script {
                    sh ''
                  }
            }
        }

        stage('Approve Deploy Prod?') {
            when {
                beforeAgent true
                anyOf {
                    branch "master"
                }
            }
            steps {
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

        stage('Deploy to Production') {
            when {
                beforeAgent true
                anyOf {
                    branch "master"
                }
                expression { return ( userInput == true ) }
            }

            agent { node { label 'master' } }
            environment { IMAGE_TAG = GIT_COMMIT.take(7) }

            steps {
                  script {
                    sh''
                  }
            }
        }
}
    post {
       always {
            cleanWs()
        }
    }
}
