pipeline {
    agent any

    environment {
        AWS_REGION = 'ap-south-1'
        AWS_ACCOUNT_ID = '386315605351'
        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        FRONTEND_REPO = 'frontend'
        BACKEND_REPO  = 'backend'
        ADMIN_REPO    = 'admin'

        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Checkout Source') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/sankar-6982/shopNow.git'
            }
        }

        stage('Verify Workspace') {
            steps {
                sh '''
                    echo "Current Directory:"
                    pwd

                    echo "Workspace Contents:"
                    ls -la
                '''
            }
        }

        stage('Build Docker Images') {
            steps {

                sh """
                    docker build \
                    -t ${FRONTEND_REPO}:${IMAGE_TAG} \
                    ./frontend
                """

                sh """
                    docker build \
                    -t ${BACKEND_REPO}:${IMAGE_TAG} \
                    ./backend
                """

                sh """
                    docker build \
                    -t ${ADMIN_REPO}:${IMAGE_TAG} \
                    ./admin
                """
            }
        }

        stage('Verify AWS Identity') {
            steps {
                sh '''
                    aws sts get-caller-identity
                 '''
            }
        }

        stage('Login to Amazon ECR') {
            steps {

                sh """
                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                """
            }
        }

        stage('Tag Docker Images') {
            steps {

                sh """
                    docker tag ${FRONTEND_REPO}:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${FRONTEND_REPO}:${IMAGE_TAG}

                    docker tag ${BACKEND_REPO}:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${BACKEND_REPO}:${IMAGE_TAG}

                    docker tag ${ADMIN_REPO}:${IMAGE_TAG} \
                    ${ECR_REGISTRY}/${ADMIN_REPO}:${IMAGE_TAG}
                """
            }
        }

        stage('Push Images to ECR') {
            steps {

                sh """
                    docker push ${ECR_REGISTRY}/${FRONTEND_REPO}:${IMAGE_TAG}

                    docker push ${ECR_REGISTRY}/${BACKEND_REPO}:${IMAGE_TAG}

                    docker push ${ECR_REGISTRY}/${ADMIN_REPO}:${IMAGE_TAG}
                """
            }
        }

        /*
        stage('Deploy to EKS') {
            steps {

                sh '''
                    kubectl apply -f k8s/
                '''
            }
        }
        */

    }

    post {

        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }

        always {

            sh '''
                docker image prune -f
            '''

            cleanWs()
        }
    }
}
