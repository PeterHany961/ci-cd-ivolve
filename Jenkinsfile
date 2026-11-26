pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'peter2976'
        IMAGE_NAME     = 'ivolve-app'
        K8S_NAMESPACE  = 'ivolve'
        DEPLOY_FILE    = 'deployment.yaml'
        APP_SVC_FILE   = 'app-service.yaml'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh "echo \$DH_PASS | docker login -u \$DH_USER --password-stdin"
                    sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Update Deployment File') {
            steps {
                sh "sed -i 's|image: .*|image: ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}|g' ${DEPLOY_FILE}"
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh "kubectl apply -f ${DEPLOY_FILE} -n ${K8S_NAMESPACE}"
                sh "kubectl apply -f ${APP_SVC_FILE} -n ${K8S_NAMESPACE}"
            }
        }

        stage('Clean Local Image') {
            steps {
                sh "docker rmi ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} || true"
            }
        }
    }
}
