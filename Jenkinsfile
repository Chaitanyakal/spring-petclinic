pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        AWS_REGION = 'ap-southeast-2'
        AWS_ACCOUNT_ID = '075810104493'
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/petclinic"
        IMAGE_TAG = "${BUILD_NUMBER}"
        EKS_CLUSTER = 'devops-project2'
        NAMESPACE = 'petclinic'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Unit Test') {
            steps {
                sh './mvnw test'
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    ./mvnw spring-boot:build-image \
                      -Dspring-boot.build-image.imageName=${IMAGE_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --ignore-status fixed \
                      --exit-code 1 \
                      ${IMAGE_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push ECR') {
            steps {
                sh '''
                    aws ecr get-login-password --region ${AWS_REGION} |
                    docker login --username AWS --password-stdin \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com

                    docker push ${IMAGE_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    aws eks update-kubeconfig \
                      --region ${AWS_REGION} \
                      --name ${EKS_CLUSTER}

                    kubectl apply -f k8s/deployment.yaml

                    kubectl set image deployment/petclinic \
                      petclinic=${IMAGE_URI}:${IMAGE_TAG} \
                      -n ${NAMESPACE}
                '''
            }
        }

        stage('Verify') {
            steps {
                sh '''
                    kubectl rollout status deployment/petclinic \
                      -n ${NAMESPACE} --timeout=300s

                    kubectl get pods -n ${NAMESPACE}
                    kubectl get svc -n ${NAMESPACE}
                '''
            }
        }
    }
}
















































































