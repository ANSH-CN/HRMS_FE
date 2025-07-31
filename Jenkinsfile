pipeline {
  agent any

  environment {
    APP = "hrms-frontend"
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONARQUBE_SERVER = "SonarEC2"  // Change this to the name of your SonarQube server configured in Jenkins
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
        script {
          sh """
            docker run --rm \
              -e SONAR_HOST_URL=${SONARQUBE_SERVER} \
              -v \$(pwd):/usr/src \
              sonarsource/sonar-scanner-cli \
              -Dsonar.projectKey=${APP} \
              -Dsonar.sources=./src \
              -Dsonar.host.url=http://:9000 \
              -Dsonar.login=your-sonarqube-token
          """
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
        sh 'docker stop ${APP} || true && docker rm ${APP} || true'
        sh 'docker run -d --name ${APP} -p 3000:80 ${IMAGE}'
      }
    }
  }

  post {
    failure {
      echo "❌ Pipeline failed. Check console output."
    }
  }
}
