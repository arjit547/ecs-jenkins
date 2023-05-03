pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECS_CLUSTER = 'clusterDev'
        ECS_SERVICE = 'react1-Service'
        IMAGE_NAME = 'ecs'
        IMAGE_TAG = 'latest'
        DOCKER_REGISTRY = '435770184212.dkr.ecr.us-east-1.amazonaws.com'
        PATH = "$PATH:/usr/local/bin"
    }
    stages {
        stage('Install ecs-cli') {
            steps {
                sh "curl https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-linux-amd64-latest -o /usr/local/bin/ecs-cli && chmod +x /usr/local/bin/ecs-cli"
                sh "sudo chmod +x /usr/local/bin/ecs-cli"
            }
        }
        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '''
                     aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 435770184212.dkr.ecr.us-east-1.amazonaws.com
                     docker build -t ecs .
                     docker tag ecs:latest 435770184212.dkr.ecr.us-east-1.amazonaws.com/ecs:latest
                     docker push 435770184212.dkr.ecr.us-east-1.amazonaws.com/ecs:latest
                    '''
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'ecr-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "ecs deploy $ECS_CLUSTER $ECS_SERVICE $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
                    sh "aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment --region $AWS_DEFAULT_REGION"
                }
            }
        }
    }
}
