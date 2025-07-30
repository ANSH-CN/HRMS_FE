pipeline {
  agent any

  environment {
    IMAGE = "cloudansh/hrms-frontend:latest"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        sh "docker build -t ${IMAGE} ."
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
          passwordVariable: 'dckr_pat_lwqdE6NsPuMBTfnKkVAaoLlZRcM'
        )]) {
          sh """
            echo \dckr_pat_lwqdE6NsPuMBTfnKkVAaoLlZRcM | docker login -u \cloudansh --password-stdin
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
