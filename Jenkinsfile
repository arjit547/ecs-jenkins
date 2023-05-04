pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECS_CLUSTER_NAME  = 'clusterDev'
        ECS_SERVICE_NAME = 'react1-Service'
        IMAGE_NAME = 'ecs'
        IMAGE_TAG = 'latest'
        ECR_REGISTRY = '435770184212.dkr.ecr.us-east-1.amazonaws.com'
        ALB_NAME  = 'app-lb'
        ALB_TARGET_GROUP_NAME = 'tg-group'
    }
    stages {
        stage('Build Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
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
                withCredentials([usernamePassword(credentialsId: 'my-aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh 'aws configure set default.region ${AWS_DEFAULT_REGION}'
                    sh "ecs-cli compose --project-name ${ECS_SERVICE_NAME} service up \
                    --launch-type FARGATE \
                    --cluster ${ECS_CLUSTER_NAME} \
                    --target-group-arn arn:aws:elasticloadbalancing:us-east-1:435770184212:targetgroup/tg-group/bb4e054c2135af79 \
                    --container-name react1-container \
                    --container-port 3000 \
                    --create-log-groups"
                    sh "aws elbv2 register-targets --target-group-arn arn:aws:elasticloadbalancing:us-east-1:435770184212:targetgroup/tg-group/bb4e054c2135af79 \
                    --targets Id=${ECS_SERVICE_NAME}:3000"

                }
            }
        }
    }
}
