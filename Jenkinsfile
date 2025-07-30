pipeline {
  agent any

  environment {
    IMAGE = "cloudansh/hrms-frontend:latest"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Docker Build') {
      steps { sh "docker build -t ${IMAGE} ." }
    }

    stage('Install Trivy') {
      steps {
        sh '''
          if ! command -v trivy &> /dev/null; then
            echo "📦 Installing Trivy..."
            curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin
          else
            echo "✅ Trivy already installed."
          fi
        '''
      }
    }

    stage('Trivy Scan') {
      steps {
        sh "trivy image --severity HIGH,CRITICAL --exit-code 1 --no-progress ${IMAGE}"
      }
    }

    stage('Docker Login & Push') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'docker-hub-creds',
          usernameVariable: 'cloudansh',
          passwordVariable: 'dckr_pat_ETOZthSkkpxzZZ8YL6bgC9JcoZo'
        )]) {
          sh """
            echo \$dckr_pat_ETOZthSkkpxzZZ8YL6bgC9JcoZo | docker login -u \$cloudansh --password-stdin
            docker push ${IMAGE}
          """
        }
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed. Check console output."
    }
  }
}
