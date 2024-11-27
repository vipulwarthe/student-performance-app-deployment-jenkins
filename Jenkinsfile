pipeline {
    agent any

    environment {
        AWS_CREDENTIALS_ID = 'aws-credentials'
        SONARQUBE_CREDENTIALS = 'sonarqube-token'
        SONARQUBE_SERVER = 'SonarQube-Server'
        ECR_REPOSITORY = 'student-performance-ml-model'
        REGION = 'us-east-1'
        CLUSTER_NAME = 'student-performance-cluster'
        SERVICE_NAME = 'student-performance-service'
        IMAGE_TAG = 'latest'
        DOCKER_IMAGE = ''
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                echo 'Workspace cleaned successfully.'
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/vipulwarthe/student-performance-ml-model-deployment-with-ECR-ECS.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(SONARQUBE_SERVER) {
                    sh '''
                    sonar-scanner \
                        -Dsonar.projectKey=student-performance \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('OWASP Dependency Check') {
            steps {
                sh '''
                dependency-check.sh \
                    --project student-performance \
                    --scan . \
                    --format ALL \
                    --out dependency-check-report
                '''
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                trivy fs . --exit-code 1 --severity HIGH,CRITICAL
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    DOCKER_IMAGE = "${env.AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                    sh """
                    docker build -t ${DOCKER_IMAGE} .
                    """
                }
            }
        }

        stage('Login to ECR') {
            steps {
                withAWS(credentials: AWS_CREDENTIALS_ID, region: REGION) {
                    sh """
                    aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${env.AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com
                    """
                }
            }
        }

        stage('Push Docker Image to ECR') {
            steps {
                sh """
                docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Deploy to ECS') {
            steps {
                withAWS(credentials: AWS_CREDENTIALS_ID, region: REGION) {
                    sh """
                    aws ecs update-service --cluster ${CLUSTER_NAME} --service ${SERVICE_NAME} --force-new-deployment
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed. Check logs for details.'
        }
    }
}
