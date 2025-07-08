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
    DOCKERFILE_BUILD_PATH = 'dockerbuild/Dockerfile'
    DOCKERHUB_USER = 'oeuvre13'
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
              -Dsonar.token=sqp_4ee12ac68f34739ce6ef24d45e5e409f34ad7f53
        """
      }
    }

    stage('Build Docker Image') {
      steps {
        sh 'docker build -f ${DOCKERFILE_BUILD_PATH} -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} .'
      }
    }

    stage('Push to Docker Hub') {
        // when {
        //     expression { return env.DOCKERHUB_USER && env.DOCKERHUB_TOKEN }
        // }
        steps {
            withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                sh """
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }


        // script {
        //     docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-creds') {
        //             def image = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
        //             image.push()
        //         }
        //     }
        // }
    }

    // stage('Provision Cloud Run with Terraform') {
    //   steps {
    //     sh 'docker build -f ${DOCKERFILE_BUILD_PATH} -t ${IMAGE_NAME}:${IMAGE_TAG} .'
    //   }
    // }
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


