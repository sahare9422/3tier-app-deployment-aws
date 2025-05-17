pipeline {
  /* Use the official Node.js image as the build environment */
  agent {
    docker {
      image 'node:18'
      /* Mount Docker socket so we can build/push images */
      args '-v /var/run/docker.sock:/var/run/docker.sock'
      label 'docker'  // ensure this agent has Docker installed
    }
  }

  environment {
    BACKEND_IMAGE          = "palash9422/3tier-backend:${env.BRANCH_NAME}"
    FRONTEND_IMAGE         = "palash9422/3tier-frontend:${env.BRANCH_NAME}"
    DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_PASSWORD'  // your Credentials ID in Jenkins
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Test Backend') {
      when {
        expression { fileExists('application-code/app-tier/package.json') }
      }
      steps {
        dir('application-code/app-tier') {
          sh 'npm install'
          sh 'npm test'
          script {
            backendImage = docker.build(env.BACKEND_IMAGE, '.')
          }
        }
      }
    }

    stage('Build Frontend') {
      when {
        expression { fileExists('application-code/web-tier/package.json') }
      }
      steps {
        dir('application-code/web-tier') {
          sh 'npm install'
          sh 'npm run build'
          script {
            frontendImage = docker.build(env.FRONTEND_IMAGE, '.')
          }
        }
      }
    }

    stage('Push Images') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', env.DOCKERHUB_CREDENTIALS_ID) {
            if (backendImage) {
              backendImage.push()
            }
            if (frontendImage) {
              frontendImage.push()
            }
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        script {
          if (backendImage) {
            sh "kubectl set image deployment/backend-deployment backend-container=${env.BACKEND_IMAGE} --namespace=default"
          }
          if (frontendImage) {
            sh "kubectl set image deployment/frontend-deployment frontend-container=${env.FRONTEND_IMAGE} --namespace=default"
          }
        }
      }
    }
  }
}
