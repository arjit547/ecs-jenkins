pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECS_CLUSTER_NAME  = 'clusterDev'
        ECS_SERVICE_NAME = 'react1-Service'
        IMAGE_NAME = 'ecs'
        IMAGE_TAG = 'latest'
        ECR_REGISTRY = '435770184212.dkr.ecr.us-east-1.amazonaws.com'
        TASK_DEFINITION_FILE = 'ecs-task-def.json'
        TARGET_GROUP_ARN = arn:aws:elasticloadbalancing:us-east-1:435770184212:targetgroup/tg-group/bb4e054c2135af79
    }
    stages {
        stage('Install ECS CLI') {
            steps {
                sh 'sudo curl https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-linux-amd64-latest -o /var/lib/jenkins/workspace/ecs-cli && sudo chmod +x /var/lib/jenkins/workspace/ecs-cli'
            }
        }
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
        stage('Create Task Definition') {
            steps {
                sh "ecs-cli compose --file $TASK_DEFINITION_FILE --project-name $IMAGE_NAME-$IMAGE_TAG create"
            }
        }
        stage('Deploy to ECS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh "ecs-cli compose --file $TASK_DEFINITION_FILE --project-name $IMAGE_NAME-$IMAGE_TAG service up --cluster $ECS_CLUSTER_NAME --service-name $ECS_SERVICE_NAME --timeout 10 --create-log-groups --target-group-arn $TARGET_GROUP_ARN --container-name $IMAGE_NAME --container-port 3000 --force-deployment"
                }
            }
        }
    }
}
