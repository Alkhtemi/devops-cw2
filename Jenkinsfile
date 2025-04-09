pipeline {
    agent any
    stages {
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t cw2-server:1.0 .'
            }
        }
        stage('Test Container') {
            steps {
                sh 'docker run -d -p 8080:8080 --name test-container cw2-server:1.0'
                sh 'curl -s http://localhost:8080 | grep "DevOps Coursework 2"'
                sh 'docker stop test-container && docker rm test-container'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "docker login -u $USER -p $PASS"
                    sh 'docker push your-dockerhub/cw2-server:1.0'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sshagent(['production-server-ssh']) {
                    sh 'ssh -o StrictHostKeyChecking=no ubuntu@production-server "kubectl rollout restart deployment cw2-deployment"'
                }
            }
        }
    }
}
