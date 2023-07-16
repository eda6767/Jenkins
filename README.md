 # Jenkins 


## Configuring Jenkins on MacOS 

<sub> 
Installing Jenkins we started from https://www.jenkins.io/download/weekly/macos/  

<br />
<br />
 
`brew install jenkins`

`ps auxww | grep jenkins`

``brew services start jenkins``



<br />

http://localhost:8080/login?from=%2F 


<br />


```
brew services info jenkins
```

<img width="512" alt="Zrzut ekranu 2023-06-18 o 15 04 47" src="https://github.com/eda6767/Jenkins/assets/102791467/8e03e6de-b1ad-4fd4-bca5-f9787134ecc9">
<br />
In case running Jenkins, first you have to stop the service by using command

```
brew services stop jenkins
ps auxww | grep jenkins 
```

Now it is time to change configuration file.

```
sudo vi /opt/homebrew/Cellar/jenkins/2.410/homebrew.mxcl.jenkins.plist
```

<img width="661" alt="Zrzut ekranu 2023-06-18 o 15 26 02" src="https://github.com/eda6767/Jenkins/assets/102791467/ce2765b1-2043-4930-952a-00a2a66c4b77">
<br />

<br />
In program arguments section I added some extra arguments:

```
<string>-DJENKINS_HOME=/Users/<user_name>/my-jenkins/jenkins-home</string>
<string>--webroot=/Users/<user_name>/my-jenkins/cache/jenkins.war</string>
<string>--pluginroot=/Users/<user_name>/my-jenkins/cache/jenkins/plugins</string>
```

Now, we can restart Jenkins service:


```
brew services restart jenkins
```

On the webside http://localhost:8080/ let's pass the password from the file:

```
cat /Users/<user>/.jenkins/secrets/initialAdminPassword
```

After this action we can finally install our suggested plugins and create new member. 
Jenkins is now ready to use.

<br />
<br />
<img width="1245" alt="Zrzut ekranu 2023-06-18 o 16 55 05" src="https://github.com/eda6767/Jenkins/assets/102791467/76ad4896-9dc6-40c3-b27f-8517006a9f91">



</sup>

# How to integrate Jenkins with Github


In order to integrate Jenkins with Github we need to Create new job based on choosen Github repository.


Notice that instead of master branch we need to use main branch, otherwise we will receive error : Couldn't find any revision to build. Verify the repository and branch configuration for this job



<img width="834" alt="Zrzut ekranu 2023-06-18 o 17 24 21" src="https://github.com/eda6767/Jenkins/assets/102791467/11002806-a9c4-4621-aa3f-4a3f490b2d6e">


<br />
<br />

<img width="595" alt="Zrzut ekranu 2023-06-18 o 17 27 49" src="https://github.com/eda6767/Jenkins/assets/102791467/de4a616d-1f42-4852-b2c5-0b60987754ea">


## Creating local DNS


## SSH to Jenkins


## Add parameter to your job

```
#!/bin/bash

NAME=$1
CONDITION_VARIABLE=$2

if [$CONDITION_VARIABLE = "true"] ; then
 echo "Hello, $NAME"
else
 echo "Goobye"
fi
```

Now, we can execute job, with given parameters

```
./script.sh Tom false
```


## Installing plugins in Jenkins


<img width="1397" alt="Zrzut ekranu 2023-06-25 o 15 54 00 1" src="https://github.com/eda6767/Jenkins/assets/102791467/ec0c40b5-d19f-48b0-b63d-8ced4dad584b">


## Create Docker container

First, we can create folder named for example jenkins_ and create Dockerfile in this folder

```
mkdir jenkins_
touch Dockerfile
```

```
docker build -t remote-image .
docker images
```

Now, we can build a container based on the image

```
docker run --name remote-container -p 80:80 remote-image
docker ps
```




Next, we create docker-compose.yml file:

```
touch docker-compose.yml
mkdir docker_home
```


docker-compose.yml file:

```
version: '3'
services:
  remote
    container_name: remote_container
    ports:
      - "8080:8080"
    volumes:
      "$PWD"/docker_home:/var/docker_home"  
    networks:
      - net
networks:
  net:        
```

Now, we need to create a virtual machine with goal to execute jobs on this particular machine - another Docker container  with SSH service, that we can connect from. 

```
mkdir centos
touch Dockerfile
```

Dockerfile looks like this:

```
FROM centos

RUN yum -y install openssh-server

RUN useradd remote_user && \
    echo "1234" | passwd remote-user --stdin && \
    mkdir /home/remote_user/.ssh && \
    chmod 700/home/remote_user/.ssh
```
Then we will generate the key:

```
ssh-keygen -f remote_key
```

It will generate 2 files. First 'remote-key', which is certificate key - this is a private key, and second 'remote-key.pub' which is a public key. Now, let's modify the Dockerfile. 

We need to add COPY instruction, which will copy a file from file system to this image. 


```
COPY remote-key.pub /home/remote_user/.ssh/authorized_keys
```

Finally Dockerfile looks like this:

```
FROM centos

RUN yum -y install openssh-server

RUN useradd remote_user && \
    echo "1234" | passwd remote-user --stdin && \
    mkdir /home/remote_user/.ssh && \
    chmod 700 /home/remote_user/.ssh

COPY remote-key.pub /home/remote_user/.ssh/authorized_keys

RUN chown remote_urser:remote_user -R /home/remote_user/.ssh/ && \
    chmod 600 /home/remote_user/.ssh/authorized_keys

RUN /usr/sbin/sshd-keygen

CMD /usr/sbin/sshd -D
```

Next step is to modify 


```
FROM centos
RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*


RUN yum -y install openssh-server
RUN yum install -y passwd

RUN useradd remote_user
RUN echo "1234" | passwd remote_user --stdin && \
    cd docker_home && \
    mkdir remote_user && \
    cd remote_user && \
    mkdir .ssh && \
    cd . && \
    cd jenkins_/centos && \
    chmod 700 /docker_home/remote_user/.ssh

COPY remote_key.pub /docker_home/remote_user/.ssh/authorized_keys

RUN chown remote_user:remote_user -R /docker_home/remote_user/.ssh/ && \
    chmod 600 /docker_home/remote_user/.ssh/authorized_keys

RUN /usr/sbin/sshd-keygen

CMD /usr/sbin/sshd -D


```

Now, we can execute:

```
docker-compose build
```


To check the running container : 

```
docker ps
```

Now, we can integrate Jenkins with our Docker container. We had previously defined remote host DNS for Jenkins containter

```
docker exec ti- jenkins bash
ssh remote_user@remote_host

```
