pipeline {
  agent any

  environment {
    IMAGE = "cloudansh/hrms-frontend:latest"
    SONARQUBE_SERVER = "SonarEC2"  // Change this to the name of your SonarQube server configured in Jenkins
    //key = "sqp_f929da7f64a4205fb15b321e2138e499af8b665d"
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

      node {
  stage('SCM') {
    checkout scm
  }
  stage('SonarQube Analysis') {
    def mvn = tool 'Default Maven';
    withSonarQubeEnv() {
      sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=hrms-frontend -Dsonar.projectName='hrms-frontend'"
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
