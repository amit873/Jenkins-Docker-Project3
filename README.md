## Jenkins-Docker-Project3
Jenkins Docker Project using Maven

1. Launch an EC2 instance for Docker host

2. Install docker on EC2 instance and start services 
  ```sh 
  yum install docker -y
  service docker status
  service docker start
  ```

3. create a new user for Docker management and add him to Docker (default) group
```sh
useradd dockeradmin
passwd dockeradmin
usermod -aG docker dockeradmin
```

4. Write a Docker file under /opt/docker

```sh
mkdir /opt/docker

### vi Dockerfile
# Pull base image
From tomcat:8-jre8 

# Maintainer
MAINTAINER "Amit-Kumar-Gupta" 

# copy war file on to container 
COPY ./webapp.war /usr/local/tomcat/webapps
```

5. Enable AWS Password Authentication and restart the sshd service

```sh
vi /etc/ssh/sshd_config
PasswordAuthentication yes
:wq!
service sshd restart
```

6. Install Pugin “Publish over SSH” 
Manage Jenkins – Manage Plugins – Available tab – Search “Publish over SSH” – Install without restart

7. Login to Jenkins console and add Docker server to execute commands from Jenkins  
Manage Jenkins --> Configure system -->  Publish over SSH --> add Docker server and credentials

```sh
SSH Server Name – Docker_Host
Hostname – 13.127.252.198 (Public IP of AWS Docker host Machine)
Username – dockeradmin
Select checkbox “Use password authentication, or use a different key”
Password – docker 
Test Configuration 
```

8. Create Jenkins job 

A) Source Code Management  
 Repository : https://github.com/amit873/Jenkins-Docker-Project3.git  
 Branches to build : */main

B) Build
 Root POM: pom.xml  
 Goals and options : clean install package  
 
C) send files or execute commands over SSH
 Name: docker_host  
 Source files	: `webapp/target/*.war`
 Remove prefix	: `webapp/target`
 Remote directory	: `//opt//docker`  
 Exec command[s]	: 
  ```sh
  docker stop amit_demo;
  docker rm -f amit_demo;
  docker image rm -f amit_demo;
  cd /opt/docker;
  docker build -t amit_demo .
  ```

D) send files or execute commands over SSH  
  Name: `docker_host`  
  Exec command	: `docker run -d --name amit_demo -p 8090:8080 amit_demo`  

9. Change ownership /opt/docker folder from root to dockeradmin 

```sh
chown -R dockeradmin:dockeradmin /opt/docker
```

10. Login to Docker host and check images and containers. (no images and containers)

```sh
docker images
docker ps
```

11. Execute Jenkins job

12. Check images and containers again on Docker host. This time an image and container get creates through Jenkins job

```sh
docker images
docker ps
```

13. Access web application from browser which is running on container

```
<docker_host_Public_IP>:8090
<docker_host_Public_IP>:8090/webapp
```
