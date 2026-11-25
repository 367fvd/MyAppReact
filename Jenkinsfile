pipeline {
  agent any

  environment {
    DOCKERHUB_USER = 'amirmijaoui'
        PATH = "C:\\Users\\amirm\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin;${env.PATH}"
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
        bat(script: 'docker rm -f client server mongodb', returnStatus: true)
        bat(script: 'docker network rm mern-network', returnStatus: true)
        bat 'docker network create mern-network'
        bat 'docker run -d --name mongodb --network mern-network mongo'
        bat 'docker run -d --name server --network mern-network -p 9000:9000 -e MONGO_URI=mongodb://mongodb:27017/mydb %DOCKERHUB_USER%/mern-server:latest'
        bat 'docker run -d --name client --network mern-network -p 3000:3000 %DOCKERHUB_USER%/mern-client:latest'
        bat 'docker ps'
      }
    }
  }
}