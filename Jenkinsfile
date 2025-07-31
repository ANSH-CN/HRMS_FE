pipeline {
  agent NewJenkinsAws


  environment {
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONAR_HOST = "http://54.172.153.126:9000"
    APP = "hrms-frontend"
  }

  tools {
    // Name must match tool configured in Jenkins > Global Tool Configuration
    sonarScanner 'Sonar'  
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
        withSonarQubeEnv('Sonar') {
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
          docker stop ${APP} || true
          docker rm ${APP} || true
          docker run -d --name ${APP} -p 3000:80 ${IMAGE}
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
