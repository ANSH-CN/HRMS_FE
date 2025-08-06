pipeline {
  agent any

  environment {
    APP = "hrms-frontend"
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONAR_HOME = tool "Sonar"
    PATH = "$HOME/.local/bin:$PATH"
  }

  stages {

    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Trivy Scan') {
      steps {
        echo '🔍 Running Trivy scan to generate report...'
        script {
          sh '''
            mkdir -p "$HOME/.local/bin"
            export PATH="$HOME/.local/bin:$PATH"

            if ! command -v trivy >/dev/null 2>&1; then
              echo "📦 Installing Trivy..."
              curl -fsSL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b "$HOME/.local/bin"
            fi

            echo "✅ Running Trivy FS scan..."
            "$HOME/.local/bin/trivy" fs --format json -o trivy-fs-report.json .
          '''
        }
      }
    }

    stage('Docker Compose Build') {
      steps {
        echo '🔧 Checking Docker Compose version...'
        script {
          sh '''
            if docker-compose version >/dev/null 2>&1; then
              echo "✅ Using docker-compose"
              docker-compose build
            elif docker compose version >/dev/null 2>&1; then
              echo "✅ Using docker compose"
              docker compose build
            else
              echo "❌ Docker Compose not found!"
              exit 1
            fi
          '''
        }
      }
    }
  }
}
