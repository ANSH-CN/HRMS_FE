pipeline {
  agent any

  tools {
    sonarQubeScanner 'SonarScanner'  // 👈 Name configured in Jenkins > Global Tools
  }

  environment {
    APP = "hrms-frontend"
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONARQUBE_TOKEN = credentials('sonar-token')  // 👈 Jenkins credential ID for Sonar token
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarQube') {  // 👈 Name configured in Jenkins > Configure System
          sh '''
            sonar-scanner \
              -Dsonar.projectKey=hrms-frontend \
              -Dsonar.sources=. \
              -Dsonar.host.url=http://localhost:9000 \
              -Dsonar.login=$SONARQUBE_TOKEN
          '''
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
      echo "❌ Pipeline failed. Check console output for errors."
    }
    success {
      echo "✅ Pipeline executed successfully!"
    }
  }
}
