# jenkins-pipelinetest

A tutorial about Continuous Integration and Continuous Delivery by Dockerize Jenkins Pipeline

This repository is a tutorial it tries to exemplify how to automatically manage the process of building, testing with the highest coverage, and deployment phases.

Our goal is to ensure our pipeline works well after each code being pushed. The processes we want to auto-manage:

    Code checkout
    Run tests
    Compile the code
    Run Sonarqube analysis on the code
    Create Docker image
    Push the image to Docker Hub
    Pull and run the image

First step, running up the services

Since one of the goals is to obtain the sonarqube report of our project, we should be able to access sonarqube from the jenkins service. Docker compose is a best choice to run services working together. We configure our application services in a yaml file as below.

docker-compose.yml

version: '3.2'
services:
  sonarqube:
    build:
      context: sonarqube/
    ports:
      - 9000:9000
      - 9092:9092
    container_name: sonarqube
  jenkins:
    build:
      context: jenkins/
    privileged: true
    user: root
    ports:
      - 8080:8080
      - 50000:50000
    container_name: jenkins
    volumes:
      - /home/jenkins:/var/jenkins_home #Remember that,if you dont want a permanent directory use  tmp directory so it is  designed to be wiped on system reboot.
      - /var/run/docker.sock:/var/run/docker.sock
    depends_on:
      - sonarqube

Paths of docker files of the containers are specified at context attribute in the docker-compose file. Content of these files as follows.

sonarqube/Dockerfile

FROM sonarqube:6.7-alpine

jenkins/Dockerfile

FROM jenkins:2.60.3

for latest image you can use :

FROM jenkins/jenkins:lts , FROM jenkinsci/blueocean 
also you can edit sonarqube docker image 

FROM sonarqube:lts

If we run the following command in the same directory as the docker-compose.yml file, the Sonarqube and Jenkins containers will up and run.

docker-compose -f docker-compose.yml up --build 

To run in background :

docker-compose -f docker-compose.yml up --build -d


Review important points of the Jenkins file

stage('Initialize'){
    def dockerHome = tool 'myDocker'
    def mavenHome  = tool 'myMaven'
    env.PATH = "${dockerHome}/bin:${mavenHome}/bin:${env.PATH}"
}

The Maven and Docker client tools we have defined in Jenkins under Global Tool Configuration menu are added to the PATH environment variable for using these tools with sh command.

stage('Push to Docker Registry'){
    withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
        pushToImage(CONTAINER_NAME, CONTAINER_TAG, USERNAME, PASSWORD)
    }
}

Note:the docker & maven tool names must be same as one we configured

During Docker credential configure give the ID name same as dockerHubAccount ,if you change modify Jenkinsfile accordingly.

Check our application is running on docker container from the local host.

docker ps

http://localhost:8090/

will see Welcome to Dockerizing Jenkins Pipeline Tutorial

Triggering a Jenkins build from a push to Github.

sonarqube integration with jenkins:

Either you can use sonarcube server running on docker in the same machine.

login into sonarcube web console with default username & password (admin&admin)

generate token from sonarcube --copy the id 

create new credentails for sonarcube with secret text & add the paste the token

on jenkins server go to configure system -give sonarcube url & credentials id 

For sonarcube scanner: Install tools by install automatically --select from maven --select version

Or else You can install sonarcube scanner in jenkins server itself:

login into jenkins docker

create sonar-scanner directory under /var/jenkins_home

wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-3.3.0.1492-linux.zip


    unzip the Sonar Scanner binary:

unzip sonar-scanner-cli-3.3.0.1492-linux.zip

update Jenkins to point to sonar-scanner binary (Manage Jenkins > Global Tool Configuration > SonarQube Scanner); you will need to uncheck ???Install automatically??? so you can explicitly set SONAR_RUNNER_HOME as /var/jenkins_home/sonar-scanner-3.3.0.1492-linux

reference : https://funnelgarden.com/sonarqube-jenkins-docker/






note: Edit sonarcube url in pom.xml


Reference : https://github.com/hakdogan/jenkins-pipeline



