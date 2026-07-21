pipeline {

    agent any

    environment {
        AWS_REGION     = 'ap-south-1'
        AWS_ACCOUNT_ID = '386315605351'
        CLUSTER_NAME   = 'demo-eks'

        ECR_REGISTRY = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"

        FRONTEND_IMAGE = 'frontend'
        BACKEND_IMAGE  = 'backend'
        ADMIN_IMAGE    = 'admin'
    }

    stages {

        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Verify Workspace') {
            steps {
                sh '''
                pwd
                ls -la
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -t ${FRONTEND_IMAGE}:${BUILD_NUMBER} ./frontend
                docker build -t ${BACKEND_IMAGE}:${BUILD_NUMBER} ./backend
                docker build -t ${ADMIN_IMAGE}:${BUILD_NUMBER} ./admin
                """
            }
        }

        stage('Login to Amazon ECR') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'sankar-aws'
                ]]) {
                    sh """
                    aws sts get-caller-identity

                    aws ecr get-login-password --region ${AWS_REGION} | \
                    docker login --username AWS \
                    --password-stdin ${ECR_REGISTRY}
                    """
                }
            }
        }

        stage('Tag Docker Images') {
            steps {
                sh """
                docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} ${ECR_REGISTRY}/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} ${ECR_REGISTRY}/${BACKEND_IMAGE}:${BUILD_NUMBER}
                docker tag ${ADMIN_IMAGE}:${BUILD_NUMBER} ${ECR_REGISTRY}/${ADMIN_IMAGE}:${BUILD_NUMBER}

                docker tag ${FRONTEND_IMAGE}:${BUILD_NUMBER} ${ECR_REGISTRY}/${FRONTEND_IMAGE}:latest
                docker tag ${BACKEND_IMAGE}:${BUILD_NUMBER} ${ECR_REGISTRY}/${BACKEND_IMAGE}:latest
                docker tag ${ADMIN_IMAGE}:${BUILD_NUMBER} ${ECR_REGISTRY}/${ADMIN_IMAGE}:latest
                """
            }
        }

        stage('Push Images to ECR') {
            steps {
                sh """
                docker push ${ECR_REGISTRY}/${FRONTEND_IMAGE}:${BUILD_NUMBER}
                docker push ${ECR_REGISTRY}/${BACKEND_IMAGE}:${BUILD_NUMBER}
                docker push ${ECR_REGISTRY}/${ADMIN_IMAGE}:${BUILD_NUMBER}

                docker push ${ECR_REGISTRY}/${FRONTEND_IMAGE}:latest
                docker push ${ECR_REGISTRY}/${BACKEND_IMAGE}:latest
                docker push ${ECR_REGISTRY}/${ADMIN_IMAGE}:latest
                """
            }
        }

        stage('Deploy to EKS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'sankar-aws'
                ]]) {

                    sh """
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${CLUSTER_NAME}

                    kubectl apply -f k8s/frontend/
                    kubectl apply -f k8s/backend/
                    kubectl apply -f k8s/admin/

                    kubectl rollout status deployment/frontend -n shopnow
                    kubectl rollout status deployment/backend -n shopnow
                    kubectl rollout status deployment/admin -n shopnow
                    """
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD Pipeline completed successfully!"
        }

        failure {
            echo "CI/CD Pipeline failed!"
        }

        always {
            cleanWs()
        }
    }
}
