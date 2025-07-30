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

    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-hub-creds',
          usernameVariable: 'DOCKER_USER',
          passwordVariable: 'DOCKER_PASS'
        )]) {
          sh '''
            echo "🔐 Logging in to Docker Hub..."
            echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

            echo "📤 Pushing image to Docker Hub: ${IMAGE}"
            docker push ${IMAGE}
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Docker image pushed successfully: ${IMAGE}"
    }
    failure {
      echo "❌ Pipeline failed. Check Jenkins console output for details."
    }
  }
}
