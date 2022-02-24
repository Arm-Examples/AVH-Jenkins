# Arm Virtual Hardware Jenkins Tutorial

This tutorial is an example of a Jenkins server running an ML application [TFL-microspeech](https://github.com/ARM-software/VHT-TFLmicrospeech) on Arm Virtual Hardware agent in the cloud.

## Start an instance

Follow the steps from this tutorial to [launch](https://github.com/ARM-software/Tool-Solutions/tree/master/mlops-cloud#amilaunch) and connect to the [Arm Virtual Hardware AMI](https://github.com/ARM-software/Tool-Solutions/tree/master/mlops-cloud#console).

When connected with SSH, type the following commands to configure the instance for the Jenkins agent (ffpmeg is only needed in our tutorial to generate different input data sets):

```
$ sudo apt update
$ sudo apt install fontconfig openjdk-11-jre ffmpeg
$ ssh-keygen -f ~/.ssh/jenkins_agent_key
$ cat ~/.ssh/jenkins_agent_key.pub >> ~/.ssh/authorized_keys
```

Copy the content of ~/.ssh/jenkins_agent_key for later.

## Start Jenkins with Docker

You can run Jenkins in a [Docker container](https://github.com/jenkinsci/docker/blob/master/README.md). In the rest of this tutorial, we are going to configure the Arm Virtual Hardware instance as an [agent](https://www.jenkins.io/doc/book/using/using-agents/).

In a terminal, clone this repository on your local machine and type the following commands:

```
$ git clone mlops-jenkins.git
$ docker pull jenkins/jenkins
$ docker run -p 8080:8080 -p 50000:50000 -v $PWD/jenkins_home:/var/jenkins_home jenkins/jenkins
```

## Configure Jenkins

Open a web browser to the following URL: [http://localhost:8080](http://localhost:8080) 

Follow the steps to configure Jenkins with default plugins. Note that the default password for the admin is displayed in the terminal running the docker instance.

Set up the user admin with a password of your choice.

## Add Jenkins plugin to plot data

Go to *Manage Jenkins* => *Jenkins CLI* and download *jenkins-cli.jar*. 

In a different terminal window, type the following commands:

```
$ java -jar ~/Downloads/jenkins-cli.jar -auth admin:avh -s http://localhost:8080/ -webSocket install-plugin Plot -deploy
$ java -jar ~/Downloads/jenkins-cli.jar -auth admin:avh -s http://localhost:8080/ -webSocket restart
```

From the web browser, refresh and log in as admin again:
http://localhost:8080
You can view the TFL-microspeech job with run data, tests reports and plots

## Configure Jenkins node

Go to *Dashboard* => *Nodes* => *Arm Virtual Hardware Instance 1* => *Configure*:

- Specify the host IP of the instance that has been launched earlier
- In *Credentials*, click on *Add* =>*Jenkins*
- - Specify *SSH username with private key*
- - Specify _ubuntu_ in *username*
- - In *Private key: Enter directly*, add and copy the key created on the AVH instance, specify the passphrase if needed
- - Add key
- Save configuration
- Click on *Launch agent* and *Trust SSH Host Key* to make the connection. The node should now appear online.

## Enable TFL-microspeech project

The TFL-microspeech project is disabled by default. Click on *Enable* to start building and testing the project. By default, a build will be launched every 10 minutes but this can be modified in the job configuration.
