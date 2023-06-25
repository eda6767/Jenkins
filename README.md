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


# Creating local DNS


# SSH to Jenkins


# Add parameter to your job



