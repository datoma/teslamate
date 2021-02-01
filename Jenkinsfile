pipeline {
  environment {
    registry = 'registry/repository'
    registryCredential = 'secret'
    dockerImage = ''
    //imageLine = 'debian:latest'
    imageLine = 'registry/repository:4'
  }

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
          dockerImage = docker.build("datoma/teslamate:latest")
        }

      }
    }

    stage('Deploy our image') {
      parallel {
        stage('Deploy our image') {
          steps {
            script {
              docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
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
        sh "docker rmi datoma/teslamate:latest"
      }
    }
  }
}
