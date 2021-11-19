pipeline{
    agent any
    tools{
        maven 'maven'
    }
    environment {
        AWS_ACCOUNT_ID="853973692277"
        AWS_DEFAULT_REGION="us-east-1" 
        IMAGE_REPO_NAME="myrepo"
        GIT_TAG = "build-$BUILD_NUMBER"
        IMAGE_TAG="build-$BUILD_NUMBER"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }

    stages{
        //Building application
        stage('Build'){
            steps{
                sh 'mvn clean package'
            }
        }

        //Creating git tag
        stage('Tagging'){
            steps{
                sshagent (credentials: ['github_ssh_key']) {
                    sh "git tag -a $GIT_TAG -m 'Version $BUILD_NUMBER'"
                    sh('git push git@github.com:umeshtyagi829/maven-project.git HEAD:$BRANCH_NAME --tag')
                
                 }
            }
        }
        
         // Building Docker images
        stage('Building Image') {
            steps{
                sh('sudo docker build  -t ${IMAGE_REPO_NAME}:${IMAGE_TAG} .')
            }
        }

        // Logging into ECR
        stage('Logging into AWS ECR') {
            steps {
                sh('aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | sudo docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com') 
            }
        }

        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
        steps{  
                sh('sudo docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG')
                sh('sudo docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}')
            }
        }
    }
    post{
        success{
            build job: 'testing_pipeline', parameters: [string(name: 'BUILD_NUMBER', value: "$BUILD_NUMBER")]
        }
    }
}


