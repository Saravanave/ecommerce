pipeline {
  agent any
  environment {
    DOCKERHUB_USER = "saravanave"
    DEV_REPO = "dev"
    PROD_REPO = "prod"
    IMAGE_NAME = "devops-build-react-app"
    SSH_USER = "ubuntu"
    EC2_HOST = "43.205.208.134"
  }
  stages {
    stage('Checkout Code') {
      steps {
        checkout scm
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
          def current_repo = (env.BRANCH_NAME == 'main') ? env.PROD_REPO : env.DEV_REPO
          def current_tag = "${env.BRANCH_NAME}-${BUILD_NUMBER}"
          sh "docker build -t ${DOCKERHUB_USER}/${current_repo}:${current_tag} ."
          sh "docker tag ${DOCKERHUB_USER}/${current_repo}:${current_tag} ${DOCKERHUB_USER}/${current_repo}:latest"
        }
      }
    }
    stage('Login to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
        }
      }
    }
    stage('Push to Docker Hub') {
      steps {
        script {
          def current_repo = (env.BRANCH_NAME == 'main') ? env.PROD_REPO : env.DEV_REPO
          def current_tag = "${env.BRANCH_NAME}-${BUILD_NUMBER}"
          sh "docker push ${DOCKERHUB_USER}/${current_repo}:${current_tag}"
          sh "docker push ${DOCKERHUB_USER}/${current_repo}:latest"
        }
      }
    }
    stage('Deploy to Server') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub_credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sshagent(['ec2-ssh-credentials']) {
            script {
              def current_repo = (env.BRANCH_NAME == 'main') ? env.PROD_REPO : env.DEV_REPO
              def current_tag = "${env.BRANCH_NAME}-${BUILD_NUMBER}"
              sh """
                ssh -o StrictHostKeyChecking=no ${SSH_USER}@${EC2_HOST} \\
                "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}; \\
                docker pull ${DOCKERHUB_USER}/${current_repo}:${current_tag}; \\
                docker stop react-app-container || true; docker rm react-app-container || true; \\
                docker run -d --name react-app-container -p 80:80 ${DOCKERHUB_USER}/${current_repo}:${current_tag}"
              """
            }
          }
        }
      }
    }
  }
}
