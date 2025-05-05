pipeline {
  agent { label 'Jenkins-Agent' }
  tools{
    jdk 'Java17'
    maven 'Maven3'
  }
  environment {
    APP_NAME = "register-app-pipeline"
    RELEASE = "1.0.0"
    DOCKER_USER = "baranishot" // Make sure to store in Jenkins credentials if possible
    DOCKER_PASS = "dockerhub" // Reference to Jenkins credentials store
    IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
    IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
  }
  stages{ 
    stage("Cleanup Workspace"){
      steps {
      cleanWs()
      }
    }
    stage("Checkout from SCM"){
      steps {
        git branch: 'main', credentialsId: 'github', url: 'https://github.com/baranishot/register-app'
      }
    }
    stage("Build Application") {
      steps {
        sh "mvn clean package"
      }
    }
    stage("Test Application"){
      steps {
        sh "mvn test"
      }
    }
    stage("SonarQube Analysis"){
      steps {
        script {
          withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token') {
            sh "mvn sonar:sonar"
          }
        }
      }
    }
    stage("Quality Gate"){
      steps {
        script {
          waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
        }
      }
    }
    stage("Build and Push Docker Image") {
      steps {
        script {
          // Using Docker Hub with the proper registry URL
          docker.withRegistry('https://hub.docker.com/v2/', DOCKER_PASS) {
            def docker_image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
            docker_image.push()
            docker_image.push('latest')  // Optionally push the 'latest' tag as well
          }
        }
      }
    }
  }

  post {
    always {
      // Clean up any temporary resources if needed
      cleanWs()
    }

    success {
      // Send success notifications if needed
      echo "Pipeline succeeded!"
    }

    failure {
      // Send failure notifications if needed
      echo "Pipeline failed!"
    }
  }
}  
