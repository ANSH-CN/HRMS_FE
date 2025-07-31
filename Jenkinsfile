pipeline {
  agent any

  environment {
    APP = "hrms-frontend"
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONARQUBE_TOKEN = credentials('sonarqube-token')  // Jenkins credential (Secret Text) for SonarQube token
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('SonarQube Analysis') {
      steps {
        // This uses the SonarQube server configuration in Jenkins (name: SonarQube)
        withSonarQubeEnv('SonarQube') {  
          sh """
            docker run --rm \\
              -v \$(pwd):/usr/src \\
              sonarsource/sonar-scanner-cli:latest \\
              -Dsonar.projectKey=hrms-frontend \\
              -Dsonar.sources=/usr/src \\
              -Dsonar.host.url=http://54.172.153.126:9000 \\
              -Dsonar.login=\$SONARQUBE_TOKEN
          """
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
