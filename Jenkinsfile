pipeline {

  agent none

  environment {
    DOCKER_IMAGE = "nnvu187/flask-docker"
  }

  stages {
    stage("Test") {
      agent {
          docker {
            image 'python:3.8-slim-buster'
            args '-u 0:0 -v /tmp:/root/.cache'
          }
      }
      steps {
        sh "pip install poetry"
        sh "poetry install"
        sh "poetry run pytest"
      }
    }

    stage("build") {
      agent { node {label 'master'}}
      environment {
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${BUILD_NUMBER}-${GIT_COMMIT.substring(0,7)}"
      }
      steps {
        withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
        }

        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
        sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
        script {
          if (GIT_BRANCH ==~ /.*master.*/) {
            sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
            sh "docker push ${DOCKER_IMAGE}:latest"
          }
        }

        //clean to save disk
        sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}"
      }
    }
  
    stage('deploy') {
      agent { node {label 'master'}}
      steps {
        sshagent(['ssh-remote-server']) {
          sh 'ssh -o StrictHostKeyChecking=no -l nnvu187 10.0.0.17 touch test.txt'
          sh "docker stop docker-flask && docker rm docker-flask"
          sh "docker run -p 5000:5000 --name docker-flask -d nnvu187/flask-docker:latest"
        }
      }
    }
  }
    post {
    success {
      echo "SUCCESSFUL"
    }
    failure {
      echo "FAILED"
    }
  }
}
