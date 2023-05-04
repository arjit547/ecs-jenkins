pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECS_CLUSTER  = 'clusterDev'
        ECS_SERVICE = 'react1-Service'
        IMAGE_NAME = 'ecs'
        IMAGE_TAG = 'latest'
        ECR_REGISTRY = '435770184212.dkr.ecr.us-east-1.amazonaws.com'
        CONTAINER_PORT = '3000'
        CPU_UNITS = '256'
        MEMORY_MB = '512'
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
                    sh "sudo apt-get update && sudo apt-get install -y python3-pip"
                    sh "pip install awscli --upgrade"
                    sh "aws ecs create-task-definition --family task --container-definitions '[{\"name\":\"task\",\"image\":\"$ECR_REGISTRY/ecs:$IMAGE_TAG\",\"essential\":true,\"portMappings\":[{\"containerPort\":$CONTAINER_PORT}],\"cpu\":$CPU_UNITS,\"memory\":$MEMORY_MB}]' > taskdefinition.json"
                    sh "aws ecs register-task-definition --cli-input-json file://taskdefinition.json"
                    sh "aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --force-new-deployment --task-definition task"
                }
            }
        }
    }
}
