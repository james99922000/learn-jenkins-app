pipeline {
    agent any

    environment{
       REACT_APP_VERSION = "1.0.$BUILD_ID"
       AWS_DEFAULT_REGION = 'us-east-1'
       AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod'
       AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service-Prod'
       AWS_ES_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod'


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
                    echo "Small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }

        stage('Build Docker image') {
            agent {
                docker{
                    image 'amazon/aws-cli'
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock:--entrypoint=''"
                    reuseNode true
                }

            }            
            
            
            steps {
                sh '''
                amazon-linux-extras install docker
                docker build -t myjenkinsapp .
                '''
            }
        }


        stage('Deploy to AWS'){
            agent {
                docker{
                    image 'amazon/aws-cli'
                    args "-u root --entrypoint=''"
                    reuseNode true
                }

            }

            steps {

                withCredentials([usernamePassword(credentialsId: 'my_aws', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ES_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable  --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }

            }
        }

    }    
}

