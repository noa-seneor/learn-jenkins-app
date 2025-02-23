pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = 'eu-west-1'
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        APP_NAME = 'learnjenkinsapp'
        AWS_DOCKER_REGISTRY = '011528270432.dkr.ecr.eu-west-1.amazonaws.com'
    }

    stages {
  stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('build docker image') {
            steps {
                sh '''
                docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                '''
            }
        }

        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            environment {
                AWS_S3_BUCKET = 'learn-jenkins-230284529'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'aws-key', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                sh '''
                    aws --version
                    # aws s3 sync build s3://$AWS_S3_BUCKET
                    yum install jq -y
                    LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws-task-definition.json | jq '.taskDefinition.revision')
                    echo "Latest Task Definition Revision: $LATEST_TD_REVISION"
                    aws ecs update-service --cluster learn-jenkins-app-cluster-prod --service LearnJenkinsApp-Service-Prod --task-definition Learning-Jenkins-TaskDefinition-Prod:$LATEST_TD_REVISION
                '''
                }
            }
        }
    }
}
