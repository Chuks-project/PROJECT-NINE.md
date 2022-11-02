## TOOLING WEBSITE DEPLOYMENT AUTOMATION WITH CONTINUOUS INTEGRATION. INTRODUCTION TO JENKINS

In previous Project 8 we introduced horizontal scalability concept, which allow us to add new Web Servers to our Tooling Website and having successfully deployed a set up with 2 Web Servers and also a Load Balancer to distribute traffic between them. It is not such a big task to manually configure two or three Web Servers. But imagine when you would need to repeat the same task over and over again adding dozens or even hundreds of servers.

DevOps is about Agility, and speedy release of software and web solutions. One of the ways to guarantee fast and repeatable deployments is Automation of routine tasks.

In this project we are going to start automating part of our routine tasks with a free and open source automation server called JENKINS. 
Jenkins is an open source self contained automation server. We will be using Jenkins free style project to automate pulling changes in our github repository, build, test and update our NFS server. We are going to utilize Jenkins CI capabilities to make sure that every change made to the source code in GitHub https://github.com/<yourname>/tooling will be automatically be updated to the Tooling Website.

### INSTALL AND CONFIGURE JENKINS SERVER
#### Step 1 – Install Jenkins server

- Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"

- Install JDK (since Jenkins is a Java-based application)


``                                                                                                                                     `   
    sudo apt update `                                                                                                                                       `
    
    sudo apt install default-jdk-headless
``

- Install Jenkins

 ``                                                                                                                             
    wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
    sudo apt update
    sudo apt-get install jenkin
``    

- Make sure Jenkins is up and running

  ``sudo systemctl status jenkins``                                                                                                   

![Jenkins running ok](https://user-images.githubusercontent.com/65022146/199357103-7dd4fa54-3006-4c72-8da2-eab344409660.png)

- By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group

- Perform initial Jenkins setup

![jenking ready](https://user-images.githubusercontent.com/65022146/199358168-a08f411f-40cc-4cbe-bb0b-826a2862b725.png)

- Then you will be asked which plugings to install – choose suggested plugins

- Once plugins installation is done – create an admin user and you will get your Jenkins server address.![Jenkins page](https://user-
![Create Admin User for Jenkings](https://user-images.githubusercontent.com/65022146/199359120-35d37181-b56c-45eb-9cd8-51e773f8995c.png)
![Jenkins is ready](https://user-images.githubusercontent.com/65022146/199359122-25759fe1-0ab9-4020-8609-fa0ce3e9037d.png)
![Jenkins page](https://user-images.githubusercontent.com/65022146/199359445-ef4efc53-794a-40ea-bfcf-3c1fb08ab9a8.png)


### Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks

- Enable webhooks in your GitHub repository settings

![Webhook configured](https://user-images.githubusercontent.com/65022146/199359715-2c84a8fc-a417-4b8c-9f61-e5eee03558bf.png)



- Go to Jenkins web console, click "New Item" and create a "Freestyle project"

- To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

- In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

- Save the configuration and let us try to run the build. For now we can only do it manually.
Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

- You can open the build and check in "Console Output" if it has run successfully

![Build one successful](https://user-images.githubusercontent.com/65022146/199364289-3fd7ff26-b7a4-4822-b58d-6420fd5ff306.png)
![Console ouptut successful](https://user-images.githubusercontent.com/65022146/199364291-93b8a2a0-0b7e-4384-b96e-530b55e92e09.png)


- But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

- Click "Configure" your job/project and add these two configurations

- Configure triggering the job from GitHub webhook by selecting

1. Github Hook for GITscm poling from the BUILD TRIGGER

2. Configure "Post-build Actions" to archive all the files 

Note that files resulted from a build are called "artifacts".

- Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch

- You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server


![New build was successful](https://user-images.githubusercontent.com/65022146/199364598-c6592c58-e524-4be8-841f-aae95089b0c8.png)
![new build console output](https://user-images.githubusercontent.com/65022146/199364595-6b940330-34a6-48da-b064-d47f52436e30.png)



## STEP 3: CONFIGURE JENKINS TO COPY FILES TO NFS SERVER VIA SSH 

### Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory

### Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH"

- Install "Publish Over SSH" plugin

- Configure the job/project to copy artifacts over to NFS server

- Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)

- Arbitrary name

- Hostname – can be private IP address of your NFS server

- Username – ec2-user (since NFS server is based on EC2 with RHEL 8)

- Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server

- Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.


![Publish over ssh successful](https://user-images.githubusercontent.com/65022146/199481019-29b9de80-d740-4a56-acb8-e8df825afdf6.png)
![configure1](https://user-images.githubusercontent.com/65022146/199481068-0130e5cf-2db3-47c8-b585-ccbcfe4eac9b.png)


### Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

- Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

``        
        SSH: Transferred 25 file(s)
        Finished: SUCCESS
``            

- This is represented in much details as seen in the screenshot below:

![Send over ssh successful](https://user-images.githubusercontent.com/65022146/199482407-0ddfb6ef-2bc2-46e7-938a-4d3594c9430f.png)

- To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file

    `cat /mnt/apps/README.md`
    
- If everything is configured successfully, you will see an image like the one below:

![Cat README md successful](https://user-images.githubusercontent.com/65022146/199483043-cf1004bd-dcdc-46de-86dd-9ff892f5632e.png)
