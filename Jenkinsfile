pipeline {
  environment { 
    registry = "registry/repository" 
    registryCredential = 'secret' 
    dockerImage = '' 
  }
  
  agent any
  stages {
    stage('') {
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
      steps { 
        script { 
          docker.withRegistry( '', registryCredential ) { 
            dockerImage.push()
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
}
