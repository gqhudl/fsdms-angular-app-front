pipeline {
  agent any
  environment {
    DOCKERHUBNAME = "dockerhubbo"
  }
  stages {
    stage('docker build & push image on build docker/build server') {
      steps {
        script {
          def REMOVE_FLAG = sh(returnStdout: true, script: "docker container ls -aq --filter name=.*fsdms-angular-app.*") != ""
          echo "REMOVE_FLAG: ${REMOVE_FLAG}"
          if(REMOVE_FLAG){
            sh 'docker container rm -f $(docker container ls -aq --filter name=.*fsdms-angular-app.*)'
          }
        }
        
        script {
          def REMOVE_FLAG = sh(returnStdout: true, script: "docker image ls -q *${DOCKERHUBNAME}/fsdms-angular-nginx*") != ""
          echo "REMOVE_FLAG: ${REMOVE_FLAG}"
          if(REMOVE_FLAG){
            sh 'docker image rm -f $(docker image ls -q *${DOCKERHUBNAME}/fsdms-angular-nginx*)'
          }
        }

        withCredentials([usernamePassword(credentialsId: "DockerHub_${DOCKERHUBNAME}", usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
          sh 'docker login -u $USERNAME -p $PASSWORD'
          sh 'docker image build -t ${DOCKERHUBNAME}/fsdms-angular-nginx:latest .'
          sh 'docker push ${DOCKERHUBNAME}/fsdms-angular-nginx:latest'
        }
      }
    }

    // the following steps should be running on deploy server which should be different with previous server normally
    // while we use the same server is just for demo purpose
    stage('docker pull image and docker run image on docker/deploy server') {
      steps {
        // docker stop/rm older containers: remove only there are containers found
        script {
          def REMOVE_FLAG = sh(returnStdout: true, script: "docker container ls -aq --filter name=.*fsdms-angular-app.*") != ""
          echo "REMOVE_FLAG: ${REMOVE_FLAG}"
          if(REMOVE_FLAG){
            sh 'docker container rm -f $(docker container ls -aq --filter name=.*fsdms-angular-app.*)'
          }
        }

        // docker rmi old images: remove only there are images found
        script {
          def REMOVE_FLAG = sh(returnStdout: true, script: "docker image ls -q *${DOCKERHUBNAME}/fsdms-angular-nginx*") != ""
          echo "REMOVE_FLAG: ${REMOVE_FLAG}"
        }

        sh 'docker run -d -p 80:80 --network fsdms-net --name fsdms-angular-app ${DOCKERHUBNAME}/fsdms-angular-nginx'
      }
    }

    stage('clean workspace') {
      steps {
        cleanWs()
      }
    }
  }
}
