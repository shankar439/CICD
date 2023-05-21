# Continuous Integration And Continuous Deployment
![imagegit](https://github.com/shankar439/Images/assets/70714976/c37d1585-a843-4dee-8262-16514413bf77)
# Overview
Jenkins as CI-CD (Continuous integration and continuous deployment) tool, scripted a Declarative pipeline for various stages 

Stage 1. Using Git to access GitHub to check out the code ,the connection between jenkins and GitHub is SSH, using Public <br>
and Private key to access private repository, 

Stage 2. Building a JAR file(Java ARchive) using Maven command # mvn clean, mvn install 

Stage 3. so, in next stage Docker can utilize the JAR file to create docker image based on Docker file.

Stage 4. push the created Docker image into ECR(AWS-Elastic Container Registry) using "View Push Command" provided by AWS.

Stage 5. As I am using t2micro instance limited on resource, I am stopping and deleting the previous Build running container.
```
docker ps -aq | xargs --no-run-if-empty docker stop | xargs --no-run-if-empty docker rm
```
"docker ps -aq" will show all running container/ all container ID. "xargs --no-run-if-empty" to suppress if no ID found by docker ps

Stage6. Using the docker image which we pushed to ECR in previous stage and running it in the same EC2 instance <br>
as Containerized application to Accomplish Software Development Lifecycle.

## create Ec2 instance and open inbound port 8080 in security group

## install jenkins, docker in ec2 instance

https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS

https://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html

## add jenkins user to docker group
```
sudo usermod -a -G docker jenkins
```
## lets restart jenkins, docker and system demon

```
sudo service jenkins restart
sudo service docker restart
sudo systemctl daemon-reload
```

## install docker and docker pipeline plugin in jenkins

## install Git in same instance and Maven using manage jenkins 
```
yum install git
```
## create repository in ECR- Elastic Container Repository

Create an IAM role with "AmazonEC2ContainerRegistryFullAccess" policy, attach to Jenkins

## Create a Pipeline Job in jenkins


```
    agent any
    tools{
        maven 'maven'
    }
    stages{
        stage('scm-git'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'provide the credential here', url: 'git@github.com:shankar439/demoSpringApp.git']]])
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean'
                sh 'mvn install'
            }
        }
        stage('buils image'){
            steps{
                script{
                    sh 'docker build -t hello-app .'
                }
            }
        }
        stage('push'){
            steps{
                script{
                    sh 'aws ecr get-login-password --regi   on ap-south-1 | docker login --username AWS --password-stdin accountNO.dkr.ecr.ap-south-1.amazonaws.com'
                    sh 'docker tag hello-app:latest accountNO.dkr.ecr.ap-south-1.amazonaws.com/aws-ecr-repo-demo-job:latest'
                    sh 'docker push accountNO.dkr.ecr.ap-south-1.amazonaws.com/aws-ecr-repo-demo-job:latest'
                }
            }
        }
        stage('cleaning containers') {
            steps {
                script{
                    sh 'docker ps -aq | xargs --no-run-if-empty docker stop | xargs --no-run-if-empty docker rm'
                }
            }
        }
        stage('Docker Run') {
            steps{
                script {
                  sh 'docker run -d -p 9090:9090 accountNO.dkr.ecr.ap-south-1.amazonaws.com/aws-ecr-repo-demo-job:latest'
                }
            }
        }


    }
}
```
# END