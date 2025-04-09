pipeline {
  agent any

  environment {
    IMAGE_NAME = "o0moh0o/cw2-server"
    IMAGE_TAG = "1.0"
    DOCKER_CREDENTIALS_ID = "dockerhub-creds"
  }

  stages {
  stage('Clone Repo') {
  steps {
    git branch: 'main', url: 'https://github.com/Alkhtemi/devops-cw2.git'
  }
}

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
      }
    }

    stage('Test Container Launch') {
      steps {
        sh '''
          docker run -d -p 9999:8081 --name test-container $IMAGE_NAME:$IMAGE_TAG
          sleep 5
          curl -s http://localhost:9999
          docker stop test-container && docker rm test-container
        '''
      }
    }

    stage('Push to DockerHub') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIALS_ID}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh '''
            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
            docker push $IMAGE_NAME:$IMAGE_TAG
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sshagent(credentials: ['prod-server-key']) {
          sh '''
            ssh -o StrictHostKeyChecking=no ubuntu@54.196.99.255 << EOF
              kubectl set image deployment/cw2-server cw2-server=$IMAGE_NAME:$IMAGE_TAG --record
            EOF
          '''
        }
      }
    }
  }
}
