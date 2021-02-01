pipeline {
  environment {
    DOCKERHUB_IMAGE_NAME = 'datoma/teslamate'
    ARTIFACTORY_IMAGE_NAME = 'datoma.jfrog.io/docker-local/teslamate'
    dockerHubImageLatest = ''
    dockerHubImagetag = ''
    artifactoryImageLatest = ''
    artifactoryImageTag = ''
    imageLine = 'datoma/teslamate:latest'
  }

  agent any
  stages {
    stage ("prepare") {
      steps {
        script {
            currentBuild.displayName = "#${BUILD_NUMBER} (DockerTag: ${DOCKER_IMAGE_TAG} - Branch: ${env.GIT_BRANCH})"
            sh 'printenv'
        }
      }
    }
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
      parallel {
        stage('push Dockerhub Tag') {
          steps {
            script {
              dockerHubImagetag = docker.build("${DOCKERHUB_IMAGE_NAME}:${DOCKER_IMAGE_TAG}")
            }
          }
        }
        stage('push Artifactory latest') {
          steps {
            script {
              artifactoryImageLatest = docker.build("${ARTIFACTORY_IMAGE_NAME}:latest")
            }
          }
        }
        stage('push Artifactory Tag') {
          steps {
            script {
              artifactoryImageTag = docker.build("${ARTIFACTORY_IMAGE_NAME}:${DOCKER_IMAGE_TAG}")
            }
          }
        }
      }
    }

    stage('Deploy and Test the latest image') {
      parallel {
         stage('Trivy') {
          steps {
            script {
              dockerHubImageLatest.run("--name trivy-client --rm -it -v /var/run/docker.sock:/var/run/docker.sock:ro datoma/trivy-server:latest trivy client --remote https://trivy.blackboards.de ${DOCKERHUB_IMAGE_NAME}:latest")
            }
          }
        }
        stage('Deploy image with latest to Dockerhub') {
          steps {
            script {
              docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                dockerHubImageLatest.push()
              }
            }
          }
        }
        stage('Deploy image with latest to Artifactory') {
          steps {
            script {
                docker.withRegistry('https://datoma.jfrog.io/artifactory', 'ArtifactoryDockerhub') {
                artifactoryImageLatest.push()
              }
            }
          }
        }
      }
    }

    stage('Deploy and Test the tagged image') {
      parallel {
         stage('Trivy') {
          steps {
            script {
              dockerHubImageLatest.run("--name trivy-client --rm -it -v /var/run/docker.sock:/var/run/docker.sock:ro datoma/trivy-server:latest trivy client --remote https://trivy.blackboards.de --ignore-unfixed --exit-code 1 --severity CRITICAL,HIGH,MEDIUM ${DOCKERHUB_IMAGE_NAME}:${DOCKER_IMAGE_TAG}")
            }
          }
        }
        stage('Deploy image with tag to Dockerhub') {
          steps {
            script {
              docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                dockerHubImagetag.push()
              }
            }
          }
        }
        stage('Deploy image with tag to Artifactory') {
          steps {
            script {
                docker.withRegistry('https://datoma.jfrog.io/artifactory', 'ArtifactoryDockerhub') {
                artifactoryImageTag.push()
              }
            }
          }
        }
      }
    }
  }

  post {
    always {
      sh "docker rmi ${DOCKERHUB_IMAGE_NAME}:latest"
      sh "docker rmi ${DOCKERHUB_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
      sh "docker rmi registry.hub.docker.com/${DOCKERHUB_IMAGE_NAME}:latest"
      sh "docker rmi registry.hub.docker.com/${DOCKERHUB_IMAGE_NAME}:${DOCKER_IMAGE_TAG}""
      sh "docker rmi ${ARTIFACTORY_IMAGE_NAME}:latest"
      sh "docker rmi ${ARTIFACTORY_IMAGE_NAME}:${DOCKER_IMAGE_TAG}"
    }
  }
}
