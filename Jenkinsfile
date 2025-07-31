pipeline {
  agent any

  environment {
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONARQUBE_SERVER = "Sonar" // This should match Jenkins global SonarQube server name
    SONAR_HOST = "http://54.172.153.126:9000"
  }

  tools {
    sonarQubeScanner 'Sonar' // This should match your SonarQube scanner tool name in Jenkins Global Tools
  }

  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }

    stage('Docker Build') {
      steps {
        sh 'docker build -t ${IMAGE} .'
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv("${SONARQUBE_SERVER}") {
          sh '''
            sonar-scanner \
              -Dsonar.projectKey=hrmsfrontend \
              -Dsonar.projectName=hrmsfrontend \
              -Dsonar.sources=. \
              -Dsonar.host.url=${SONAR_HOST}
          '''
        }
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
            docker push ${IMAGE}
          '''
        }
      }
    }

    stage('Docker Run') {
      steps {
        sh '''
          docker stop hrms-frontend || true
          docker rm hrms-frontend || true
          docker run -d --name hrms-frontend -p 3000:80 ${IMAGE}
        '''
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed. Check console output."
    }
  }
}
