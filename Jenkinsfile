pipeline {
    agent any
    tools {
        dockerTool 'docker'
    }
    environment {
        // IMAGE_NAME = 'cloud1111/jenkins-flask-app-demo'
        // IMAGE_TAG = "${IMAGE_NAME}:${env.BUILD_NUMBER}"
        IMAGE_NAME = "cloud1111/jenkins-flask-app-demo:${BUILD_NUMBER}"
        AWS_REGION = 'us-east-1'
    }
    stages {
        stage('Test') {
            steps {
                echo "Test Stage"
                sh "whoami"
                // sh "ls -al ~/.kube/config"
            }
        }
        stage('Login to docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                sh 'docker login -u ${USERNAME} -p ${PASSWORD}'}
                echo 'Login successfully'
            }
        }
        stage('Build Docker Image')
        {
            steps
            {
                sh 'sudo docker build -t ${IMAGE_NAME} .'
                echo "Docker image build successfully"
                sh "sudo docker images"
            }
        }
        stage("TRIVY"){
            steps{
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh "trivy image --no-progress --exit-code 1 --severity MEDIUM,HIGH,CRITICAL --format table ${IMAGE_NAME}"
                 }   
            }
        }
        stage('Push Docker Image')
        {
            steps
            {
                sh 'sudo docker push ${IMAGE_NAME}'
                echo "Docker image push successfully"
            }
        }
    }
}
