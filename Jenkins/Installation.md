# Install Jenkins the easy way
Install Jenkins the easy way with docker

## Pre-Requisites
To install any version of jenkins:
- Install Docker (`https://docs.docker.com/get-docker/`)
- After Installing pull the latest image of jenkins from docker hub

        docker pull jenkins/jenkins:lts-jdk11

- Preferably choose the lighter version of the image to reduce load of RAM

        docker pull jenkins/jenkins:alpine-jdk11


## Run Jenkins:
- Run jenkins on `http://localhost:8080` with one easy command

        docker run --rm -it --name jenkins-app -p 8080:8080 -p 50000:50000 --restart=on-failure -v jenkins_home:/var/jenkins_home jenkins/jenkins:alpine-jdk11

- To bash into the jenkins container:

        docker exec -it jenkins-app bash

- After running jenkins, browse to `http://localhost:8080` and Unlock Jenkins

        sudo cat /var/lib/jenkins/secrets/initialAdminPassword

        sudo docker exec jenkins-app cat /var/jenkins_home/secrets/initialAdminPassword

- Then follow along with the remaining steps on the Jenkins UI


## Customization:
- Build your custom jenkins image by creating a Dockerfile:

        FROM jenkins/jenkins:2.346.2-jdk11
        USER root
        RUN apt-get update && apt-get install -y lsb-release
        RUN curl -fsSLo /usr/share/keyrings/docker-archive-keyring.asc \
            https://download.docker.com/linux/debian/gpg
        RUN echo "deb [arch=$(dpkg --print-architecture) \
            signed-by=/usr/share/keyrings/docker-archive-keyring.asc] \
            https://download.docker.com/linux/debian \
            $(lsb_release -cs) stable" > /etc/apt/sources.list.d/docker.list
        RUN apt-get update && apt-get install -y docker-ce-cli
        USER jenkins
        RUN jenkins-plugin-cli --plugins "blueocean:1.25.5 docker-workflow:1.28"

- Build the Dockerfile:

        docker build -t myjenkins-blueocean:2.346.2-1 .