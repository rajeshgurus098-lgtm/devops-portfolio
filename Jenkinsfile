pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        AWS_ACCOUNT_ID = '510724490791'
        REPOSITORY_NAME = 'devopsportfolio'
    }

    stages {

        stage('Clone') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                docker build -t jenkins-ecr-push .
                '''
            }
        }

        stage('Login ECR') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-creds']]) {
                    bat '''
                        aws ecr get-login-password --region %AWS_REGION% > password.txt
                        type password.txt | docker login --username AWS --password-stdin %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com
                        del password.txt
                    '''
                }
            }
        }

        stage('Tag Image') {
            steps {
                bat '''
                docker tag jenkins-ecr-push:latest %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%REPOSITORY_NAME%:latest
                '''
            }
        }

        stage('Push Image') {
            steps {
                bat '''
                docker push %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%REPOSITORY_NAME%:latest
                '''
            }
        }

        stage('Deploy') {
            steps {
                bat '''
                docker rm -f devops-portfolio-site 2>nul
                docker run -d -p 8081:80 --name devops-portfolio-site %AWS_ACCOUNT_ID%.dkr.ecr.%AWS_REGION%.amazonaws.com/%REPOSITORY_NAME%:latest
                '''
            }
        }
    }
}
