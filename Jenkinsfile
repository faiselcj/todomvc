pipeline {

    agent any

    environment {
        AWS_REGION = "us-east-1"
        ACCOUNT_ID = "093359840674"
        ECR_REPO = "react-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        ECR_URI = "${ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/faiselcj/todomvc.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                dir('examples/react') {
                    sh 'npm install'
                }
            }
        }

        stage('Build React') {
            steps {
                dir('examples/react') {
                    sh 'npm run build'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                dir('examples/react') {
                    sh '''
                        docker build -t react-app:${IMAGE_TAG} .
                    '''
                }
            }
        }
		stage('Login ECR') {
			steps {
				withCredentials([
					[$class: 'AmazonWebServicesCredentialsBinding',
					 credentialsId: 'aws cred']
				]) {
					sh '''
					aws sts get-caller-identity
					aws ecr get-login-password \
					--region ${AWS_REGION} | \
                    docker login \
                    --username AWS \
					--password-stdin ${ECR_URI}
					'''
				}
			}
		}

        stage('Login ECR') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'aws cred',
                        usernameVariable: 'AWS_ACCESS_KEY_ID',
                        passwordVariable: 'AWS_SECRET_ACCESS_KEY'
                    )
                ]) {
                    sh '''
                        aws ecr get-login-password \
                        --region ${AWS_REGION} \
                        | docker login \
                        --username AWS \
                        --password-stdin ${ECR_URI}
                    '''
                }
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker tag react-app:${IMAGE_TAG} ${ECR_URI}:${IMAGE_TAG}
                    docker push ${ECR_URI}:${IMAGE_TAG}
                '''
            }
        }

        stage('Deploy ECS') {
            steps {
                sh '''
                    aws ecs update-service \
                    --cluster react-cluster \
                    --service react-service \
                    --force-new-deployment \
                    --region ${AWS_REGION}
                '''
            }
        }
    }
}
