pipeline {
    agent any

    environment {
       AWS_DEFAULT_REGION = 'eu-west-1'
       AWS_ECS_CLUSTER = 'learn-jenkins-app-cluster-prod'
       AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
       AWS_ECS_TD_PROD = 'Learning-Jenkins-TaskDefinition-Prod'

    }

    stages {


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
                    aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                    aws ecs wait seek-service-stable --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD
                '''
                }
            }
        }

        // stage('Build') {
        //     agent {
        //         docker {
        //             image 'node:18-alpine'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             ls -la
        //             node --version
        //             npm --version
        //             npm ci
        //             npm run build
        //             ls -la
        //         '''
        //     }
        // }
    }
}
