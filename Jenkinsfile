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

    stage('Check Trivy Scan') {
      steps {
        echo 'Check Trivy Scan To Generate Report...'
        script {
          sh '''
            # install trivy into user directory if missing
            mkdir -p "$HOME/.local/bin"
            export PATH="$HOME/.local/bin:$PATH"

            if ! command -v trivy >/dev/null 2>&1; then
              echo "Installing trivy into $HOME/.local/bin"
              curl -fsSL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b "$HOME/.local/bin"
            fi

            # verify and run scan
            trivy fs --format json -o trivy-fs-report.json .
          '''
        }
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
      archiveArtifacts artifacts: 'trivy-fs-report.json', fingerprint: true
      archiveArtifacts artifacts: 'reports/dependency-check-report.json', fingerprint: true
    }

    failure {
      echo "❌ Pipeline failed. Check logs."
    }
  }
}
