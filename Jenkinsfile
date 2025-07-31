pipeline {
  agent any

  environment {
    APP = "hrms-frontend"
    IMAGE = "cloudansh/hrms-frontend:latest"
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t ${IMAGE} .'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh 'trivy image --severity CRITICAL,HIGH ${IMAGE} || true'
      }
    }

    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-hub-creds',
          usernameVariable: 'DOCKER_USERNAME',
          passwordVariable: 'DOCKER_PASSWORD'
        )]) {
          sh '''
            echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
            docker push ${IMAGE}
          '''
        }
      }
    }
  }
}
