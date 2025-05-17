pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "palash9422/3tier-backend:${env.BRANCH_NAME}"
        FRONTEND_IMAGE = "palash9422/3tier-frontend:${env.BRANCH_NAME}"
        DOCKERHUB_CREDENTIALS_ID = 'DOCKER_HUB_PASSWORD'
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
                        backendImage = docker.build(env.BACKEND_IMAGE)
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
                        frontendImage = docker.build(env.FRONTEND_IMAGE)
                    }
                }
            }
        }

        stage('Push Images') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', env.DOCKER_HUB_PASSWORD) {
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

        // stage('Deploy Backend') {
        //     when {
        //         expression { fileExists('application-code/app-tier/package.json') }
        //     }
        //     steps {
        //         sh "kubectl set image deployment/backend-deployment backend-container=${env.BACKEND_IMAGE} --namespace=default"
        //     }
        // }

        // stage('Deploy Frontend') {
        //     when {
        //         expression { fileExists('application-code/web-tier/package.json') }
        //     }
        //     steps {
        //         sh "kubectl set image deployment/frontend-deployment frontend-container=${env.FRONTEND_IMAGE} --namespace=default"
        //     }
        // }
    }
}
