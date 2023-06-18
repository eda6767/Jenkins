 # Jenkins 

<sub><sup> https://www.jenkins.io/download/weekly/macos/ </sub></sup>

<sub> 
  
`brew install jenkins`

`ps auxww | grep jenkins`

``brew services start jenkins``


<img width="547" alt="Zrzut ekranu 2023-06-18 o 14 58 53" src="https://github.com/eda6767/Jenkins/assets/102791467/eb7bb6fe-f0b0-4a9a-ba0d-a357004d182c">


<br />
<sub><sup>  http://localhost:8080/login?from=%2F  </sub></sup>
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
<string>-DJENKINS_HOME=/Users/edytakorba/my-jenkins/jenkins-home</string>
<string>--webroot=/Users/edytakorba/my-jenkins/cache/jenkins.war</string>
<string>--pluginroot=/Users/edytakorba/my-jenkins/cache/jenkins/plugins</string>
```

</sup>
