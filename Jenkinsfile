pipeline {
   agent none 
   environment {
       DOCKERHUB_CREDENTIALS = credentials('DockerLogin')
   }
   stages {
       stage('Build'){
           agent {
               docker {
                   image 'node:lts-buster-slim'
               }
           }	
           steps {
               sh 'npm install'
           }
       }
        stage('Build Docker Image and push to Docker Registry'){
           agent {
               docker {
                   image 'docker:dind'
                   args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
               }
           }
           steps {
               sh 'docker build -t auliaarifin/nodejsgoof:0.1 .'
               sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
               sh 'docker push auliaarifin/nodejsgoof:0.1'
           }
        }
        stage('Deploy Docker Image'){
            agent {
                docker {
                    image 'kroniak/ssh-client'
                    args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
               withCredentials([sshUserPrivateKey(credentialsId: "DeploymentSSHKey", keyFileVariable: 'keyfile')]) {
                   //sh 'ssh -i ${keyfile} -o StrictHostKeyChecking=no osboxes@10.10.10.10 "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"'
                   sh 'ssh -i ${keyfile} -o StrictHostKeyChecking=no osboxes@10.10.10.10 docker pull auliaarifin/nodejsgoof:0.1'
                   sh 'ssh -i ${keyfile} -o StrictHostKeyChecking=no osboxes@10.10.10.10 docker rm --force mongodb'
                   sh 'ssh -i ${keyfile} -o StrictHostKeyChecking=no osboxes@10.10.10.10 docker run --detach --name mongodb -p 27017:27017 mongo:3'
                   sh 'ssh -i ${keyfile} -o StrictHostKeyChecking=no osboxes@10.10.10.10 docker rm --force nodejsgoof'
                   sh 'ssh -i ${keyfile} -o StrictHostKeyChecking=no osboxes@10.10.10.10 docker run -it --detach -p 3001:3001 --name nodejsgoof --network host auliaarifin/nodejsgoof:0.1'
               } 
            }
        } 
   }
}
