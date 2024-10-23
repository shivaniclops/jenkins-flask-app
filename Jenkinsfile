pipeline {
    agent {
        node {label "UAT"}
    }
    
    tools {
        dockerTool 'docker' 
    }
    
    environment {
        IMAGE_NAME = "cloud1111/jenkins-flask-app-demo:${BUILD_NUMBER}"
        AWS_REGION = 'us-east-1'
    }
    
    stages {
        stage('Test') {
            steps {
                echo "Test Stage"
                sh "whoami"
                sh "ls -al ~/.kube/config"
            }
        }
        stage('Login to docker hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-cred', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                sh 'sudo docker login -u ${USERNAME} -p ${PASSWORD}'}
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
    //     stage('Deploy to EKS Cluster') {
    //         steps {
    //             withCredentials([aws(credentialsId: 'aws-cred', region: AWS_REGION)]) {
    //             // sh "sed -i 's#TAG/${IMAGE_TAG}#g' deployment.yaml"
    //             sh "sed -i 's#TAG#${BUILD_NUMBER}#g' deployment.yaml"
    //             sh " aws eks update-kubeconfig --region us-east-1 --name CastAI-POC-EKS-Cluster"
    //             sh "/home/ubuntu/bin/kubectl apply -f deployment.yaml -n jenkins"
    //             sh "/home/ubuntu/bin/kubectl apply -f service.yaml -n jenkins"
    //             }
    //             echo "Deployed to EKS Cluster"
    //         }
        // }
    }
}
