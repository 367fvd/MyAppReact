pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'amirmijaoui'
  }

  stages {
    stage('Build serveur') {
      steps {
        sh 'docker build -t $DOCKERHUB_USER/mern-server:latest ./server'
      }
    }

    stage('Build client') {
      steps {
        sh 'docker build -t $DOCKERHUB_USER/mern-client:latest ./client'
      }
    }

    stage('Push Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'U', passwordVariable: 'P')]) {
          sh 'echo $P | docker login -u $U --password-stdin'
          sh 'docker push $DOCKERHUB_USER/mern-server:latest'
          sh 'docker push $DOCKERHUB_USER/mern-client:latest'
        }
      }
    }

    stage('Lancer les conteneurs') {
      steps {
        sh '''
          docker rm -f client server mongodb || true
          docker network inspect mern-network >/dev/null 2>&1 || docker network create mern-network
          docker run -d --name mongodb --network mern-network mongo
          docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mydb $DOCKERHUB_USER/mern-server:latest
          docker run -d --name client --network mern-network -p 3000:3000 $DOCKERHUB_USER/mern-client:latest
          docker ps
        '''
      }
    }
  }
}