pipeline {
    agent any

    environment {
       AWS_DEFAULT_REGION = 'eu-west-1'

    }

    stages {


        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
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
                    cat aws-task-definition.json
                    aws ecs register-task-definition --cli-input-json file://aws-task-definition.json
                    aws ecs update-service --cluster learn-jenkins-app-cluster-prod --service LearnJenkinsApp-Service-Prod --task-definition Learning-Jenkins-TaskDefinition-Prod:2
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
