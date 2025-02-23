pipeline {
    agent any

    environment {
       AWS_DEFAULT_REGION = 'eu-west-1'

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
                sh 'docker build -t my-jenkins-app .'
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
                    aws ecs wait seek-service-stable --cluster learn-jenkins-app-cluster-prod --service LearnJenkinsApp-Service-Prod
                '''
                }
            }
        }
    }
}
