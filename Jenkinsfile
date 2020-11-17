pipeline {
  agent any
  stages {
    stage('Clone') {
      steps {
        git(url: 'https://github.com/datoma/teslamate.git', branch: 'master')
      }
    }

    stage('Building our image') {
      steps {
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }

      }
    }

    stage('Deploy our image') {
      parallel {
        stage('Deploy our image') {
          steps {
            script {
              docker.withRegistry( '', registryCredential ) {
                dockerImage.push()
              }
            }

          }
        }

        stage('Test') {
          steps {
            writeFile file: 'anchore_images', text: imageLine
            anchore name: 'anchore_images'
          }
        }

      }
    }

    stage('Cleaning up') {
      steps {
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }

  }
  environment {
    registry = 'registry/repository'
    registryCredential = 'secret'
    dockerImage = ''
    //imageLine = 'debian:latest'
    imageLine = 'registry/repository:4'
  }
}
