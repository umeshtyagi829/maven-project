pipeline{
    agent any
    tools{
        maven 'maven'
    }
    stages{
        stage('Build'){
            steps{
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image'){
            steps{
                sh 'sudo docker build -t java-mvn:0.1 .'
            }
        }
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
