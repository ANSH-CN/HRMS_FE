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
      }
    }

    stage('Trivy Scan') {
      steps {
        sh '''
          mkdir -p trivy-reports contrib

          # Download HTML report template if not exists
          curl -sSL -o contrib/html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl

          # Generate JSON report
          trivy image --severity CRITICAL,HIGH --format json -o trivy-reports/trivy-report.json ${IMAGE}

          # Generate HTML report
          trivy image --severity CRITICAL,HIGH --format template --template "@contrib/html.tpl" -o trivy-reports/trivy-report.html ${IMAGE}
        '''
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
        sh '''
          docker-compose down || true
          docker-compose rm -f || true
          docker-compose up -d
        '''
      }
    }
  }

  post {
    always {
      echo "📦 Archiving Trivy reports..."
      archiveArtifacts artifacts: 'trivy-reports/*', fingerprint: true
    }

    failure {
      echo "❌ Pipeline failed. Check console output."
    }
  }
}
