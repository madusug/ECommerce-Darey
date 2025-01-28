# E-Commerce Website with Jenkins

A technology consulting firm is adopting a cloud architecture for its software applications. As a DevOps Engineer, my task is to design and implement a robust CI/CD pipeline using Jenkins to automate the deployment of a web application. The goal is to achieve continuous integration, continuous deployment, and ensure the scalability and reliability of the applications.

## Jenkins Server Setup - Configure Jenkins server for CI/CD pipeline automation

Goals:

1. Install Jenkins on a dedicated server
2. Set up necessary plugins (Git, Docker)
3. Configure Jenkins with required security measures

### 1. Jenkins Installation

The first step in this process is to install Jenkins. To do this, I provisioned an Ubuntu instance on AWS and did the following:

- SSH into the instance using vscode
- Update the ubuntu instance to make sure everything is up to date, install dependencies and Jenkins:

```
sudo apt-get update && sudo apt-get upgrade
sudo apt install default-jdk-headless
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
/etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
sudo systemctl status jenkins
```

**Error while installing Jenkins**

I ran into an error while trying to install Jenkins. See details below:

```
W: GPG error: https://pkg.jenkins.io/debian-stable binary/ Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 5BA31D57EF5975CA
E: The repository 'https://pkg.jenkins.io/debian-stable binary/ Release' is not signed.
N: Updating from such a repository can't be done securely, and is therefore disabled by default.
N: See apt-secure(8) manpage for repository creation and user configuration details.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Package jenkins is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

E: Package 'jenkins' has no installation candidate
```

***How I corrected this error***

I went on the Jenkins site to check what the installation instructions were `https://www.jenkins.io/doc/book/installing/linux/`. I found the following syntax and used it which worked:

```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

When this worked, I saw that I could also install Java:

```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
openjdk version "17.0.13" 2024-10-15
OpenJDK Runtime Environment (build 17.0.13+11-Debian-2)
OpenJDK 64-Bit Server VM (build 17.0.13+11-Debian-2, mixed mode, sharing)
```

But since I already did this in a previous step, I didn't use it.

The next step was for me to enable, start and check the status of Jenkins

```
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```

Now that Jenkins is installed, I opened port 8080 on my inbound port on my ubuntu instance. This is under Security groups. Doing this will allow me access jenkins on port 8080.
To access the Jenkins UI, I used: <jenkins-ip-address>:8080

### 2. Docker and Git Installation on Ubuntu Instance where Jenkins is Installed

**Git**

Git is a free and open-source version control system that tracks changes to code and allows users to collaborate on projects. With the help of Git, Jenkins can easily pull source code from any git repository that the Jenkins build node can access.

To install Git, I used: `sudo apt install git -y`

**Docker**

Here, I will be used Docker alongside Jenkins to create consistent and isolated environments for building, testing and deploying this web application across the development and production environment.

Steps to install Docker:

1. Create a file named docker.sh
2. Vim into the file and paste the script below:

```
sudo apt-get update -y
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository to Apt sources:
echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update && apt-get upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl status docker
```

3. Save and close the file
4. run `sh docker.sh`. We have successfully installed docker

5. run `sudo usermod -aG docker jenkins`. This will give jenkins access to docker on the instance where jenkins is installed

### 3. Jenkins Configuration

Upon accessing Jenkins, it shows that Jenkins will need to be unlocked. To ensure Jenkins is securely set up by the admin, a password was written to the log and this file on the server:
`/var/lib/jenkins/secrets/initialAdminPassword`

To access the password, I used:
`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`

![JenkinsUnlock](./img/1%20Jenkins%20Unlock.jpg)

After unlocking Jenkins,I selected "Install suggested plugins".

![Jenkins Plugins](./img/2%20Jenkins%20Plugins.jpg)

After the plugins are unlocked, I created the Admin User (Username, Password, Full name and E-mail address), after which, I started Jenkins.

## Source Code Management Repository Integration

Next, I connected Jenkins to the version control system for source code management. This let me subscribe to events happening on Github.
To do this, I:

1. Integrated Jenkins with GitHub
2. Configured webhooks for automatic triggering of Jenkins builds

### Integrate Jenkins with Github

Next, I connected Jenkins to Github using webhooks. On Github, go to settings, click on Webhooks, click Add webhook. Under Payload URL input <ip-address>:8080/github-webhook.

![Githook](./img/3%20Githook.jpg)

## Jenkins Freestyle Jobs for Build and Unit Tests

Here I created a Jenkins freestyle job for building a web application and running unit tests.

I selected new item on Jenkins, chose 'freestyle' project, and gave it a name without spaces. To configure the freestyle project, I chose 'Git' as my Source Code Management and used my github repository web url as the Repository URL on Jenkins. I also changed the Branch Specifier to */main. Under Build Triggers, I chose GitHub hook trigger for GITScm polling and saved it.

![freestyle](./img/4%20Freestyle.jpg)

## Jenkins Pipeline for Web Application

I also developed a a pipeline job on Jenkins for running web applications.

