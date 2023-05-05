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
        EXECUTION_ROLE_ARN = 'arn:aws:iam::435770184212:role/ecsTaskExecutionRole'
        SUBNET_ID_1 = 'subnet-0bed575f58c89b793'
        SUBNET_ID_2 = 'subnet-0ce4c422484528322'
        SECURITY_GROUP_ID = 'sg-0171390e40b1bb16a'
        TASK_DEF_REVISION = sh (script: "aws ecs describe-task-definition --task-definition $TASK_DEF_FAMILY --query 'taskDefinition.revision'", returnStdout: true).trim()
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
        stage('Create ECS Task Definition for Fargate with VPC') {
            steps {
                script {
                    def taskDefJson = """{
                      "family": "${TASK_DEF_FAMILY}",
                      "networkMode": "awsvpc",
                      "executionRoleArn": "${EXECUTION_ROLE_ARN}",
                      "requiresCompatibilities": [
                        "FARGATE"
                      ],
                      "cpu": "${TASK_DEF_CPU}",
                      "memory": "${TASK_DEF_MEMORY}",
                      "containerDefinitions": [
                        {
                          "name": "${TASK_DEF_CONTAINER_NAME}",
                          "image": "${TASK_DEF_IMAGE}",
                          "cpu": ${TASK_DEF_CPU},
                          "memory": ${TASK_DEF_MEMORY},
                          "portMappings": [
                            {
                              "containerPort": 3000,
                              "protocol": "tcp"
                            }
                          ]
                        }
                      ],
                      "networkMode": "awsvpc", 
                      "tags": [
                        {
                            "key": "Name",
                            "value": "myterraformvpc"
                        }
                      ]
                    }"""
                    
                    def result = sh (script: "aws ecs register-task-definition --cli-input-json '${taskDefJson.replaceAll('"', '\\"')}'", returnStdout: true)
                    def taskDefArn = sh(script: "echo \${result} | jq -r '.taskDefinition.taskDefinitionArn'", returnStdout: true).trim()
                    def taskDefRevision = sh(script: "echo \${taskDefArn} | cut -d: -f6", returnStdout: true).trim()
                    env.TASK_DEF_REVISION = taskDefRevision
                }
            }
        }
        stage('Deploy to ECS') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-creds', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                  sh "ecs deploy --service $ECS_SERVICE_NAME --cluster $ECS_CLUSTER_NAME --image $TASK_DEF_IMAGE:$TASK_DEF_REVISION --target-group-arn arn:aws:elasticloadbalancing:us-east-1:435770184212:targetgroup/tg-group/bb4e054c2135af79 --container-name $TASK_DEF_CONTAINER_NAME --container-port 3000"
                }
            }
        }
    }
}
