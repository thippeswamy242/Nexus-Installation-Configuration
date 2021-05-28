# Nexus-Installation-Configuration

# Nexus Repository Manager 2  –  System Requirements

## Supported Versions
Sonatype fully supports versions of Nexus Professional for one year after their release date. Older releases are supported on a best effort basis and the release dates of these versions are listed in our download archives1

## Host Operating System

Any Windows, Linux, or Macintosh operating system that can run a supported Java version will work. Other operating systems may work, but they are not tested by Sonatype.

The most widely used operating system for Nexus is Linux and therefore customers should consider it the best tested platform.

### Dedicated Operating System User Account

Unless you are testing the repository manager or running it only for personal use, a Nexus Repository Manager user account specific to the operating system is strongly recommended for running.
The Nexus Repository Manger process user is typically named nexus and must be able to create a valid shell.

***Note: As a security precaution, do not run Nexus as the root user.***
### Adequate File Handles
The Nexus Repository Manager process may want to consume more file handles than the default value allowed by your operating system.
We recommend increasing the available file handles2 for Nexus preemptively to avoid potential errors at runtime.

1 https://help.sonatype.com/display/NXRM2/Download+Archives+-+Repository+Manager+
2 https://support.sonatype.com/hc/en-us/articles/213465218-The-nexus-log-file-is-full-of-too-many-open-files-exceptions-how-can-I-fix-this

## Java
Sonatype Nexus is a Java server application that requires specific versions to operate.

Nexus Version 2.14.11+   onwards you can use The most recent version of Java 8 available.This version and after will no longer boot on Java 7.

Note : We strongly suggest using the latest compatible release version of Java available.

## CPU
Nexus Repository Manager performance is primarily bounded by IO (disk and network) rather than CPU.  So any reasonably modern 2-4 core CPU will generally be sufficient for normal use.
## Memory
The default JRE max heap size of Nexus Professional is 768Mb, and the codebase of Nexus Repository Manager will consume approximately another 1GB.  So factoring in operating system overhead you will need at least 4GB of RAM, assuming no other large applications are running on the machine.For very large Nexus Repository Manager instances you will need to bump up the maximum heap space. If you do this you should increase RAM on the system accordingly.
## Disk
Sonatype Nexus, installed, consumes less than 100MB. The bulk of disk space will be held by your deployed and proxied artifacts, as well as any search indexes. This is highly installation specific.


# Installation steps for Redhat with t2.medium instance

### Install Java
$ sudo yum list | grep java-1.8
$ sudo yum install java-1.8.0-openjdk -y
$ sudo java --version

### Install Nexus

***Download Nexus from official website***
https://help.sonatype.com/repomanager3/download

***Go to /opt directory and download nexus package from official website***

$ cd /opt
$ sudo wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz

***Extract tar file***

$ sudo tar -xzf <name of package>

***Rename Nexus***
  
$ sudo mv <package name> nexus3

***Change owner ship of the file***
  
$ sudo useradd nexus
  
$ sudo chown -R nexus:nexus nexus3/ sonatype-work/
  
$ ls -ltr
  
### Run Nexus as a service
  
It requires when you boot up your system it automatically start the nexus service if we run as a service.  

***Go to Nexus official website you will find the steps to configure***
https://help.sonatype.com/repomanager3/installation/run-as-a-service
 
$ cd /opt/nexus3/bin
  
***Step 1***
  
$ sudo vim nexus.rc
  
run_as_user="nexus"

***Step 2***
  
 Create a soft link
  
 $ sudo ln -s /opt/nexus3/bin/nexus /etc/init.d/nexus
  
 $ cd /etc/init.d
 $ sudo chkconfig --add nexus
 $ sudo chkconfig --level 345 nexus on
  
 $ service nexus start
  
 $ service nexus status
  
 ### Now login to nexus server 
  https://<Public ip:8081>
  
 ## Now create a Repository in Nexus
  
  *** Create repo ---> maven2hosted
  
  Name : Demoapp-release
  
  Version : release
  
  & everything is default
  
  create repositry ***
  
  
 ## Now configure in Jenkins
  
  Install a plugin called "NexusArtifactUploader"
  
  ### Now create jenkins pipeline job
  
  Go to snipper generator
  
  Select  nexusArtifactUploader:NexusArtifactUploader
  
  And give details as i mentioned below
  
  ```Nexus Version Nexus3
     Protocol: HTTP
     Nexus url : <private ip:8081>
     credentials : <Nexus credentials>
     GroupId : in.javahome    -------> you can found this in pom.xml file
     Version : 1.0.0
     Repositry : demoapp-release   ------> Which you created in nexus
     Add (Select)
  
     Artifacts
        ArtifactId : simpleapp  (From pom file)
        Type : war
        Classifier : Not required
        File : target/simple-app-1.0.0.war
  ```
  
 Generate script copy the code and paste it in the stage of Jenkins file to upload artifact to nexs server
  
  ### Jenkinsfile
  
  ```stage('Upload war to Nexus') {
      steps {
        nexusArtifactUploader artifacts: [
            [
              artifactId: 'simple-app',
              classifier: '',
              file: 'target/simple-app-1.0.0.war',
              type: 'war'
            ]
          ],
          credentialsId: 'nexus3',
          groupId: 'in.javahome',
          nexusurl: '172.21.14.301:8081',
          nexusVersion: 'nexus3'
        protocol: 'http',
          repositry: 'demoapp-release',
          version: '1.0.0'
      }
    }
  ```
  
 #### And build your pipeline job
  
### Note: In above generated script the Versions are hardcoded. If we need to change version in pom.xml file again we need to go Jenkinsfile and there also we need to change version to reflect it. To overcome this we are following one more method in below steps.
  
  
## Read versions form maven pom file
  
##### In order to dynamically read versions from pom.xml. We need to install Plugin called "PipelineUtilitySteps"
  
 In google search Jenkins read pom version and select pipeline utility steps and select readMavenPom.
  
  ###
  
  ```stage('Upload war to Nexus') {
      steps {
        script {
          def mavenPom = readMavenPom file: 'pom.xml'
          nexusArtifactUploader artifacts: [
              [
                artifactId: 'simple-app',
                classifier: '',
                file: "target/simple-app-${mavenPom.version}.war",
                type: 'war'
              ]
            ],
            credentialsId: 'nexus3',
            groupId: 'in.javahome',
            nexusurl: '172.21.14.301:8081',
            nexusVersion: 'nexus3'
          protocol: 'http',
            repositry: 'demoapp-release',
            version: "${mavenPom.version}"

        }

      }
  ```
  
  
  
  
  
  
  
  
 


  














