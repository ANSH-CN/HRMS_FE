pipeline {
  agent any

  environment {
    APP = "hrms-frontend"
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONARQUBE_ENV = "SonarEC2"  // Use the exact name configured in Jenkins > Manage Jenkins > Configure System
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('SonarQube Scan') {
      steps {
        withSonarQubeEnv("${SONARQUBE_ENV}") {
          sh 'sonar-scanner'
        }
      }
    }

    stage('Docker Compose Build') {
      steps {
        sh 'docker-compose build'
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
          '''
        }
      }
    }

    stage('Deploy Using Compose') {
      steps {
        sh 'docker-compose down || true'
        sh 'docker-compose up -d'
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed. Check console output."
    }
  }
}
