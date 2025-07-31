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

    stage('Verify Docker Compose') {
      steps {
        sh 'docker-compose version || docker compose version || echo "❌ Docker Compose not found"'
      }
    }

    stage('Docker Compose Build') {
      steps {
        sh 'docker-compose build'
        // or use: sh 'docker compose build'
      }
    }

    stage('Trivy Scan') {
      steps {
        sh "trivy image --severity CRITICAL,HIGH ${IMAGE} || true"
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
            docker-compose push
            # or: docker compose push
          '''
        }
      }
    }

    stage('Deploy Using Compose') {
      steps {
        sh 'docker-compose down || true'
        sh 'docker-compose up -d'
        // or: docker compose down/up
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed. Check console output."
    }
  }
}
