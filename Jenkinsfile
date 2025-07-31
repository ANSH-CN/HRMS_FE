pipeline {
  agent any

  environment {
    APP = "hrms-frontend"
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONAR_HOME = tool "Sonar"
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
          echo "🔍 Running Trivy Filesystem Scan..."
          mkdir -p trivy-fs-reports contrib

          docker run --rm aquasec/trivy --version

          # JSON report
          docker run --rm -v $(pwd):/project aquasec/trivy fs \
            --severity HIGH,CRITICAL \
            --format json \
            --output /project/trivy-fs-reports/trivy-fs-report.json \
            /project || true

          # HTML report
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

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("Sonar") {
          sh '''
            echo "🔎 Starting SonarQube analysis..."
            $SONAR_HOME/bin/sonar-scanner \
              -Dsonar.projectName=hrms \
              -Dsonar.projectKey=hrms \
              -Dsonar.sources=. \
              -Dsonar.language=js \
              -Dsonar.sourceEncoding=UTF-8
          '''
        }
      }
    }

    stage('OWASP Dependency Check') {
      steps {
        echo '⚙️ Running OWASP Dependency Check...'
        sh 'mkdir -p reports'
        dependencyCheck additionalArguments: '--scan . --format JSON --out reports --project "hrms-project"', odcInstallation: 'dc'
      }
    }

    stage('📊 Publish OWASP Report') {
      steps {
        echo "📄 Publishing OWASP Dependency Check JSON Report..."
        dependencyCheckPublisher pattern: '**/dependency-check-report.json'
      }
    }
  }

  post {
    always {
      echo "📦 Archiving Reports..."
      archiveArtifacts artifacts: 'trivy-fs-reports/*', fingerprint: true
      archiveArtifacts artifacts: 'reports/dependency-check-report.json', fingerprint: true
    }

    failure {
      echo "❌ Pipeline failed. Check logs."
    }
  }
}
