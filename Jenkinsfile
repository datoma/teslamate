pipeline {
  environment {
    DOCKERHUB_IMAGE_NAME = 'datoma/teslamate'
    ARTIFACTORY_IMAGE_NAME = 'datoma.jfrog.io/docker-local/teslamate'
    dockerImageLatest = ''
    dockerImagetag = ''
    imageLine = 'datoma/teslamate:latest'
  }

  agent any
  stages {
    //stage ("Prompt for input") {
    //  steps {
    //    script {
    //      env.DOCKER_IMAGE_TAG = input message: 'Please enter the docker image tag', parameters: [string(defaultValue: '', description: '', name: 'DockerImageTag')]
    //      echo "docker tag: ${env.DOCKER_IMAGE_TAG}"
    //    }
    //  }
    //}

    stage('Clone') {
      steps {
        git(url: 'https://github.com/datoma/teslamate.git', branch: 'master')
      }
    }

    stage('Building our image') {
      steps {
        script {
          dockerHubImageLatest = docker.build("${DOCKERHUB_IMAGE_NAME}:latest")
        }
      }
    }
    
    stage('Tagging our image') {
      steps {
        script {
          dockerImagetag = docker.tag("${DOCKERHUB_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}")
          artifactoryImageLatest = docker.tag ("${ARTIFACTORY_IMAGE_NAME}:latest")
          artifactoryImageTag = docker.tag ("${ARTIFACTORY_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}")
        }
      }
    }

    stage('Deploy our image') {
      parallel {
         stage('Trivy') {
          steps {
            script {
              dockerHubImageLatest.run("--name trivy-client --rm -it -v /var/run/docker.sock:/var/run/docker.sock:ro datoma/trivy-server:latest trivy client --remote https://trivy.blackboards.de ${DOCKERHUB_IMAGE_NAME}:latest -f json")
            }
          }
        }
        stage('Deploy our image') {
          steps {
            script {
              docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                dockerHubImageLatest.push()
              }
            }
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
