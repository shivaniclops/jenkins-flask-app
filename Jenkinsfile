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
                    sh "sudo trivy image --no-progress --exit-code 1 --severity MEDIUM,HIGH,CRITICAL --format table ${IMAGE_NAME}"
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
        stage('Update Deployment File GitOps') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    cleanWs()
                    sh """
                        git clone https://${GITHUB_TOKEN}@github.com/shivaniclops/flask-manifest.git
                        sed -i "s|cloud1111/jenkins-flask-app-demo:.*|cloud1111/jenkins-flask-app-demo:${BUILD_NUMBER}|g" deployment.yaml
                        git config user.email "shivani.d@cloudifyops.com" ## replace with your github useremail
                        git config user.name "shivaniclops"            ## replace with your github usernamename
                        git add deployment.yaml
                        git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                        git push https://${GITHUB_TOKEN}@github.com/shivaniclops/flask-manifest.git
                    """
                    cleanWs()
                }
            }
       }
    }
}
