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
          docker run --rm -v $(pwd):/project aquasec/trivy fs \
            --severity HIGH,CRITICAL \
            --format table \
            /project > trivy-fs-reports/trivy-fs-report.txt || true
        '''
      }
    }

    stage('Docker Compose Build') {
      steps {
        sh 'docker-compose build'
      }
    }

    // ... your other stages ...
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
