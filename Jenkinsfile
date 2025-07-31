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

    stage('Trivy Filesystem Scan') {
      steps {
        sh '''
          echo "🔍 Running Trivy FS scan..."
          mkdir -p trivy-fs-reports

          # Optional: check Trivy version
          docker run --rm aquasec/trivy --version

          # Plain text report
          docker run --rm -v $(pwd):/project aquasec/trivy fs \
            --severity HIGH,CRITICAL \
            --format table \
            /project > trivy-fs-reports/trivy-fs-report.json || true

          # Optional: HTML report using Trivy template
          mkdir -p contrib
          curl -sSL -o contrib/html.tpl https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl

          docker run --rm -v $(pwd):/project -v $(pwd)/contrib:/contrib aquasec/trivy fs \
            --severity HIGH,CRITICAL \
            --format template \
            --template "@/contrib/html.tpl" \
            /project > trivy-fs-reports/trivy-fs-report.html || true
        '''
      }
    }

    stage('Docker Compose Build') {
      steps {
        sh 'docker-compose version || docker compose version || echo "❌ Docker Compose not found"'
        sh 'docker-compose build || docker compose build'
      }
    }

    // Add more stages here (Trivy image scan, Docker push, Deploy, etc.)
  }

  post {
    always {
      echo "📦 Archiving Trivy FS scan report..."
      archiveArtifacts artifacts: 'trivy-fs-reports/*', fingerprint: true
    }

    failure {
      echo "❌ Pipeline failed. Check logs."
    }
  }
}
