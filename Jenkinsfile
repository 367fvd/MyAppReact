pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'amirmijaoui'
  }

  stages {
    stage('Build serveur') {
      steps {
        bat 'docker build -t %DOCKERHUB_USER%/mern-server:latest ./server'
      }
    }

    stage('Build client') {
      steps {
        bat 'docker build -t %DOCKERHUB_USER%/mern-client:latest ./client'
      }
    }

    stage('Push Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'U', passwordVariable: 'P')]) {
          bat 'docker login -u %U% -p %P%'
          bat 'docker push %DOCKERHUB_USER%/mern-server:latest'
          bat 'docker push %DOCKERHUB_USER%/mern-client:latest'
        }
      }
    }

    stage('Lancer les conteneurs') {
      steps {
        bat '''
          docker rm -f client server mongodb
          docker network rm mern-network
          docker network create mern-network
          docker run -d --name mongodb --network mern-network mongo
          docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mydb %DOCKERHUB_USER%/mern-server:latest
          docker run -d --name client --network mern-network -p 3000:3000 %DOCKERHUB_USER%/mern-client:latest
          docker ps
        '''
      }
    }
  }
}