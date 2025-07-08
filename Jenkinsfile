pipeline {
  agent any
  tools {
    maven 'Maven 3.8.8'       
    jdk 'Temurin JDK 17'
  }
 
  environment {
    SONARQUBE_SERVER = 'SonarQube' 
    IMAGE_NAME = 'cicd-challenge'
    IMAGE_TAG = 'latest'
  }
 
  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/oeuvre13/cicd-challenge', branch: 'master'
      }
    }
 
    stage('Unit Test & Coverage') {
      steps {
        sh 'mvn package'
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
        }
      }
    }
 
    stage('Static Code Analysis (SAST) via Sonar') {
      steps {
        sh """
            mvn clean compile sonar:sonar \
              -Dsonar.projectKey=cicd-challenge \
              -Dsonar.projectName='cicd-challenge' \
              -Dsonar.host.url=http://sonarqube:9000 \
              -Dsonar.token=sqp_672c2c47e7624583caa799cd452cc82afbc85de9
        """
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .'
      }
    }

    stage('Push to Docker Hub') {
        when {
            expression { return env.DOCKERHUB_USER && env.DOCKERHUB_TOKEN }
        }
        steps {
        script {
            docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
                    def image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                    image.push()
                }
            }
        }
    }
  }
 
  post {
    success {
      echo "Pipeline berhasil 🚀"
    }
    failure {
      echo "Pipeline gagal 💥"
    }
  }
}


