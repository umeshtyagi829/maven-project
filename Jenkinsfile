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
       // stage('GIT TAG'){
        //    steps{
          //      sshagent (credentials: ['github-ssh-key']) {
            //        sh "git tag -a $GIT_TAG -m 'Version $BUILD_NUMBER'"
              //      sh('git push git@github.com:umeshtyagi829/maven-project.git HEAD:$BRANCH_NAME --tag')
                
                //}
           // }
        //}

        //Loging into ECR
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | sudo docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
                 
            }
        }

        // Building Docker images
        stage('Building image') {
            steps{
            script {
                dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
            }
        }

        // Uploading Docker images into AWS ECR
        stage('Pushing to ECR') {
        steps{  
            script {
                    sh "sudo docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                    sh "sudo docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
            }
            }
        }
        //Deployiing
        stage('Deploy'){
            steps{
                sh 'sudo docker rm -f java-mvn-app'
                sh 'sudo docker run --rm -dp 4444:8080 --name java-mvn-app java-mvn:0.1'
            }
        }
    }
    post{
        success{
            archiveArtifacts artifacts: '**/target/*.war', followSymlinks: false
        }
    }
}
