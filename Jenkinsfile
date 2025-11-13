pipeline {
    agent any

    environment {
        APP_NAME = "studentmarkservice"
        APP_NAMESPACE = "${APP_NAME}-ns"
        IMAGE_NAME = "${APP_NAME}-image"
        IMAGE_TAG = "${BUILD_NUMBER}${BUILD_NUMBER}"
        APP_PORT = 8100
        NODE_PORT = 30081
        REPLICA_COUNT = 2
    }


    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/somanadha/studentmarkservice.git'
            }
        }

        stage('Cleanup Old Container (if exists)') {
            steps {
                sh '''
                    if [ "$(docker ps -aq -f name=${APP_NAME})" ]; then
                        echo "Container exists — stopping & removing..."
                        docker stop ${APP_NAME} || true
                        docker rm ${APP_NAME} || true
                    else
                        echo "No old container found."
                    fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f Dockerfile ."
            }
        }

        stage('Run Container') {
            steps {
                sh "docker run -d --name ${APP_NAME} -p 8100:8100 ${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }

        stage('K8s Deployment') {
            steps {
                script {
                    withEnv(["KUBECONFIG=$HOME/.kube/config"]) {

                        sh "envsubst < k8s/namespace-template.yaml > k8s/namespace.yaml"
                        sh "envsubst < k8s/deployment-template.yaml > k8s/deployment.yaml"
                        sh "envsubst < k8s/service-template.yaml > k8s/service.yaml"

                        sh "kubectl apply -f k8s/namespace.yaml --validate=false"
                        sh "kubectl apply -f k8s/deployment.yaml --validate=false"
                        sh "kubectl apply -f k8s/service.yaml --validate=false"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✔ Completed: Checkout → Build → Deploy"
        }
        failure {
            echo "❌ Pipeline Failed"
        }
    }
}
