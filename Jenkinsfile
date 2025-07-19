pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        IMAGE_NAME = "omegavlg/diplom-test-app"
    }

    stages {
        stage('Debug Info') {
            steps {
                script {
                    echo "BRANCH_NAME: ${env.BRANCH_NAME}"
                    echo "GIT_COMMIT: ${env.GIT_COMMIT}"
                    echo "Is tag: ${env.BRANCH_NAME.startsWith("v")}"
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    def imageTag = env.BRANCH_NAME.startsWith("v") ? env.BRANCH_NAME : "latest"

                    echo "Building Docker image with tag: ${imageTag}"

                    sh "docker build -t ${IMAGE_NAME}:${imageTag} ."
                    sh "echo ${DOCKERHUB_CREDENTIALS_PSW} | docker login -u ${DOCKERHUB_CREDENTIALS_USR} --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${imageTag}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { env.BRANCH_NAME.startsWith("v") }
            }
            steps {
                script {
                    echo "Deploying image ${IMAGE_NAME}:${env.BRANCH_NAME} to Kubernetes"

                    sh """
                        sed -i 's|image: .*|image: ${IMAGE_NAME}:${env.BRANCH_NAME}|' k8s/deployment.yml
                        cat k8s/deployment.yml
                        kubectl apply -f k8s/deployment.yml
                        kubectl rollout status deployment diplom-test-app || true
                        kubectl get deployment diplom-test-app -o jsonpath='{.spec.template.spec.containers[*].image}'
                    """
                }
            }
        }
    }

    post {
        success {
            echo '✅ Сборка и деплой выполнены успешно.'
        }
        failure {
            echo '❌ Ошибка при сборке или деплое.'
        }
    }
}
