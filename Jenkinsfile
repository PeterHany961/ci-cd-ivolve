pipeline {
    agent {
        kubernetes {
            label 'docker-agent'
            defaultContainer 'docker'
        }
    }

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
                container('docker') {
                    sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} ."
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                        sh "echo \$DH_PASS | docker login -u \$DH_USER --password-stdin"
                        sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER}"
                    }
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
                sh """
                    if command -v kubectl >/dev/null 2>&1; then
                        kubectl apply -f ${DEPLOY_FILE} -n ${K8S_NAMESPACE}
                        kubectl apply -f ${APP_SVC_FILE} -n ${K8S_NAMESPACE}
                    else
                        echo "kubectl is not installed in the current container. Add kubectl or a kubectl container to deploy."
                    fi
                """
            }
        }

        stage('Clean Local Image') {
            steps {
                container('docker') {
                    sh "docker rmi ${DOCKERHUB_USER}/${IMAGE_NAME}:${BUILD_NUMBER} || true"
                }
            }
        }
    }
}
