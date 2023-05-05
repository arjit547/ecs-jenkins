pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECS_CLUSTER_NAME  = 'clusterDev'
        ECS_SERVICE_NAME = 'react1-Service'
        IMAGE_NAME = 'ecs'
        IMAGE_TAG = 'latest'
        ECR_REGISTRY = '435770184212.dkr.ecr.us-east-1.amazonaws.com'
        TASK_DEF_FAMILY = 'task'
        TASK_DEF_CPU = '256'
        TASK_DEF_MEMORY = '512'
        TASK_DEF_CONTAINER_NAME = 'react1-container'
        TASK_DEF_IMAGE = "$ECR_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
    }
    stages {
        stage('Install ECS CLI') {
            steps {
                sh 'sudo curl https://amazon-ecs-cli.s3.amazonaws.com/ecs-cli-linux-amd64-latest -o /usr/local/bin/ecs-cli && sudo chmod +x /usr/local/bin/ecs-cli' 
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
        stage('Delete ECS Task Definition') {
            steps {
                script {
                    def taskDefList = sh(script: "aws ecs list-task-definitions --family-prefix $TASK_DEF_FAMILY --status ACTIVE --query 'taskDefinitionArns' --output text", returnStdout: true).trim()
                    for (def taskDefArn in taskDefList.split()) {
                        sh "aws ecs deregister-task-definition --task-definition $taskDefArn"
                    }
                }
            }
        }
        stage('Create ECS Task Definition') {
            steps {
                script {
                    def taskDefJson = """
                        {
                            \"family\": \"$TASK_DEF_FAMILY\",
                            \"containerDefinitions\": [
                                {
                                    \"name\": \"$TASK_DEF_CONTAINER_NAME\",
                                    \"image\": \"$TASK_DEF_IMAGE\",
                                    \"cpu\": $TASK_DEF_CPU,
                                    \"memory\": $TASK_DEF_MEMORY
                                }
                            ]
                        }
                    """
                    sh "aws ecs register-task-definition --cli-input-json '$taskDefJson'"
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                   sh "ecs deploy $ECS_SERVICE_NAME --cluster $ECS_CLUSTER_NAME --image $ECR_REGISTRY/$IMAGE_NAME:$IMAGE_TAG"
                }
            }
        }
    }
}
