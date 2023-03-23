pipeline {
  agent any

  environment {
    DOCKERHUB_CREDENTIALS = credentials('docker-hub-credentials')
    REMOTE_SERVER = '3.72.17.245'
    REMOTE_USER = 'ubuntu'            
  }

  // Fetch code from GitHub

  stages {
    stage('checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/zinchenko-ihor/JavaWebApp'

      }
    }

   // Build Java application

    stage('Maven Build') {
      steps {
        sh 'mvn clean install'
      }

     // Post building archive Java application

      post {
        success {
          archiveArtifacts artifacts: '**/target/*.jar'
        }
      }
    }

  // Test Java application

    stage('Maven Test') {
      steps {
        sh 'mvn test'
      }
    }

   // Build docker image in Jenkins

    stage('Build Docker Image') {

      steps {
        sh 'docker build -t javawebapp:latest .'
        sh 'docker tag javawebapp jumper93/javawebapp:latest'
      }
    }

   // Login to DockerHub before pushing docker Image

    stage('Login to DockerHub') {
      steps {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u    $DOCKERHUB_CREDENTIALS_USR --password-stdin'
      }
    }

   // Push image to DockerHub registry

    stage('Push Image to dockerHUb') {
      steps {
        sh 'docker push jumper93/javawebapp:latest'
      }
      post {
        always {
          sh 'docker logout'
        }
      }

    }

   // Pull docker image from DockerHub and run in EC2 instance 

    stage('Deploy Docker image to AWS instance') {
      steps {
        script {
          sshagent(credentials: ['awscred']) {
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker stop javaApp || true && docker rm javaApp || true'"
      sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker pull palakbhawsar/javawebapp'"
          sh "ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_SERVER} 'docker run --name javaApp -d -p 8081:8081 palakbhawsar/javawebapp'"
          }
        }
      }
    }
  }
}
