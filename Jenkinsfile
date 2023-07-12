pipeline{
    agent {
        label 'E2Enode'
    }

    stages{
    	stage('Clear Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('checkout'){
            steps{
                git 'https://github.com/iamvickyav/spring-boot-docker-tomcat.git'
            }
        }
        stage('Maven Build'){
            steps{
              sh 'sudo mvn clean package'
            }
        }
        
        stage('Create our own docker image'){
            steps{
               sh 'sudo docker rm -f simplewebapp'
               sh 'sudo docker build -t npaluri2/simplewebapp:${BUILD_TAG} . '
            }
        }
        stage('Push the Docker Image'){
            steps{
               withCredentials([string(credentialsId: 'Docker_Hub_Password', variable: 'Dockerhub')]) {
                sh 'sudo docker login -u npaluri2 -p ${Dockerhub}'
               
}
                 sh 'sudo docker push npaluri2/simplewebapp:${BUILD_TAG}'
            }
            
        }
        stage('Deploy in DEV ENV'){
            steps{
               sh 'sudo docker rm -f mywebapp'
               sh 'sudo docker run -d -p 8080:8080 --name mywebapp npaluri2/simplewebapp:${BUILD_TAG}'
            }
        }
        stage('Deploy in Q&A Env'){
            steps{
                sshagent(['d4657350-8de9-4863-9e98-ee60dcd79e8f']) {
                sh 'ssh -o StrictHostKeyChecking=no ec2-user@65.2.69.85 sudo docker rm -f mywebapp'
                sh 'ssh ec2-user@65.2.69.85 sudo docker run -d -p 8080:8080 --name mywebapp npaluri2/simplewebapp:${BUILD_TAG}'
}
               
            }
        }
        stage('Q&T Testing Env'){
            steps{
                retry(10){
               sh 'sudo curl --silent http://65.2.69.85:8080/sample/ | grep Welcome'      
                }
                
            }
        }
        stage('Approve'){
            steps{
                input(message:"Approve to Prod Env ?")
            }
        }
        stage('Deploy in Prod_Env'){
            steps{
                sshagent(['d4657350-8de9-4863-9e98-ee60dcd79e8f']) {
                sh 'ssh -o StrictHostKeyChecking=no ec2-user@13.234.186.17  sudo docker rm -f mywebapp'
                sh 'ssh ec2-user@13.234.186.17 sudo docker run -d -p 8080:8080 --name mywebapp npaluri2/simplewebapp:${BUILD_TAG}'
}
               
            }
        }
        
        
    }
}
