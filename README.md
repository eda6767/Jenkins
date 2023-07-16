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
<br />
Notice that instead of master branch we need to use main branch, otherwise we will receive error : Couldn't find any revision to build. Verify the repository and branch configuration for this job



<img width="834" alt="Zrzut ekranu 2023-06-18 o 17 24 21" src="https://github.com/eda6767/Jenkins/assets/102791467/11002806-a9c4-4621-aa3f-4a3f490b2d6e">


<br />
<br />

<img width="595" alt="Zrzut ekranu 2023-06-18 o 17 27 49" src="https://github.com/eda6767/Jenkins/assets/102791467/de4a616d-1f42-4852-b2c5-0b60987754ea">




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

## Installing Jenkins on Docker

For this purpose we are gonna use Jenkins image, which has already all configuration - source : https://hub.docker.com/r/jenkins/jenkins 

```
docker pull jenkins/jenkins:lts-jdk11
```

Let's create a directory for projekt and for Jenkins: 

```
mkdir jenkins_
cd jenkins_
mkdir jenkins
cd jenkins
```

Now we have to create compose docker file

```
vi docker-compose.yml
```

```
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - "8080:8080"
    volumes:
      - $PWD/jenkins_home:/var/jenkins_home
    networks:
      - net
networks:
  net:
```


```
mkdir jenkins_home
```

Now we have to create service reading the docker-compose files. 

```
docker-compose up -d
docker ps
docker logs -f jenkins
```

Next step it to unlock Jenkins through website. The password you can find typing

```
docker logs -f jenkins

```

<img width="641" alt="Zrzut ekranu 2023-07-16 o 13 31 49" src="https://github.com/eda6767/Jenkins/assets/102791467/64a8f7df-347f-45b4-a728-cef8dac2da99">


## Installing plugins in Jenkins


<img width="1397" alt="Zrzut ekranu 2023-06-25 o 15 54 00 1" src="https://github.com/eda6767/Jenkins/assets/102791467/ec0c40b5-d19f-48b0-b63d-8ced4dad584b">


## Create Docker container for SSH
Now, we need to create a virtual machine with goal to execute jobs on this particular machine - another Docker container  with SSH service, that we can connect from. First, we can 
<br />
create folder under jenkins_/jenkins : centos and create Dockerfile in this folder

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

<br />
<br />

It will generate 2 files. First 'remote-key', which is certificate key - this is a private key, and second 'remote-key.pub' which is a public key. Now, let's modify the Dockerfile. We need to add COPY instruction, which will copy a file from file system to this image. 
<br />
<br />

```
COPY remote-key.pub /home/remote_user/.ssh/authorized_keys
```

Finally Dockerfile looks like this:

```
FROM centos
RUN cd /etc/yum.repos.d/
RUN sed -i 's/mirrorlist/#mirrorlist/g' /etc/yum.repos.d/CentOS-*
RUN sed -i 's|#baseurl=http://mirror.centos.org|baseurl=http://vault.centos.org|g' /etc/yum.repos.d/CentOS-*


RUN yum -y install openssh-server
RUN yum install -y passwd

RUN useradd remote_user && \
    echo "1234" | passwd remote_user --stdin && \
    mkdir /home/remote_user/.ssh && \
    chmod 700 /home/remote_user/.ssh

COPY remote_key.pub /home/remote_user/.ssh/authorized_keys

RUN chown remote_user:remote_user -R /home/remote_user/.ssh/ && \
    chmod 600 /home/remote_user/.ssh/authorized_keys

RUN ssh-keygen -A

CMD /usr/sbin/sshd -D

```
<br />
<br />
Let's take back to our JENKINS container, and modify the docker-compose.yml file. Let's add new service called remote_host which will be used for connection via SSH to another container.
<br />
<br /> 

Out file should look like this:

```
version: '3'
services:
  jenkins:
    container_name: jenkins
    image: jenkins/jenkins
    ports:
      - "8080:8080"
    volumes:
      - $PWD/jenkins_home:/var/jenkins_home
    networks:
      - net
  remote_host:
    container_name: remote-host
    image: remote-host
    build:
      context: centos
    networks:
      - net
networks:
  net:    
```


Now, we can execute:


```
docker-compose build
```
<img width="989" alt="Zrzut ekranu 2023-07-16 o 16 01 43" src="https://github.com/eda6767/Jenkins/assets/102791467/54823d35-5e78-4bff-a6b5-f6de7bad1e7c">

<br />
Now, we can list our images:

```
docker images
```
<img width="481" alt="Zrzut ekranu 2023-07-16 o 16 02 09" src="https://github.com/eda6767/Jenkins/assets/102791467/1f41457b-17b9-4888-8891-aea68e2f4b43">

<br />
<br />

Now, we can build a container based on the image
<br />


```
docker-compose up -d
docker ps
```

<br />
To check the running container : 

```
docker ps
```

<img width="1185" alt="Zrzut ekranu 2023-07-16 o 16 13 14" src="https://github.com/eda6767/Jenkins/assets/102791467/b4c6eb80-badf-42a7-8a17-dae547203444">
<br />
<br />
Now, we can integrate Jenkins with our Docker container. We had previously defined remote host DNS for Jenkins containter and we can try to connect from jenkins container to remote_host container, using a password from Dockerfile

```
docker exec -it 3518434374fe_jenkins bash
ssh remote_user@remote_host
```

<br />
Another way to connect is by using remote-key, without a password:

```
cd centos
docker cp remote-key 3518434374fe_jenkins:/tmp/remote-key
docker exec -it 3518434374fe_jenkins bash
cd /tmp/
ls
ssh -i remote-key remote_user@remote_host
```



<img width="1391" alt="Zrzut ekranu 2023-07-16 o 12 11 46" src="https://github.com/eda6767/Jenkins/assets/102791467/80bd439d-e0d4-41a8-a765-bb709578ccc6">


Before that we need to create credentials for remote_user and with generated previously private key.


<img width="1328" alt="Zrzut ekranu 2023-07-16 o 12 20 27" src="https://github.com/eda6767/Jenkins/assets/102791467/ae7238f8-3a99-4257-8f01-932a817d1211">
