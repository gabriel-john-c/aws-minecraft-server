# aws-minecraft-server
How to host your own Minecraft server using AWS EC2 Linux instance

## Introduction

Inspired by Julien Bras and his [article](https://sidoine.org/how-to-run-a-minecraft-server-on-aws-for-less-than-3-usd-a-month/) about running a minecraft server for his son, I took it upon myself to learn how to run a minecraft server using AWS features when a handful of my own friends wanted to revisit Minecraft in our adulthood. After only knowing how to previously run servers using Windows machines and portforwarding, I decided I would use this opportunity to learn about AWS and run the server on an EC2 linux instance.  For a full list of references I used for this, please visit the [Sources](https://github.com/gabriel-john-c/aws-minecraft-server/edit/main/README.md#sources) section

## Tools used
A quick run down for which tools we use and how it plays into the general architecture of this service

- AWS
  - EC2
    - For running an Amazon Linux instance that will host the server   
  - Lambda function
    - Hosting 2 main functions to start and stop the server
  - Simple Email Service
    - For running a server once we get an email to a specified email recipient
  - CloudWatch
    - For scheduling events that will trigger the stop function at a defined interval
 
- Minecraft (Java)
  - Fabric server that the service runs off of

## Build Breakdown
Half of the name is to craft, so lets get to building...

### EC2

#### Creating the Instance

For the initial set up, we want to launch an instance. Select the 'Amazon Linux 2 AMI' type. Select the x86 variant. I went with an Amazon Linux instance with t2.large type. I wanted the server to have at least 8 GB of RAM since it would be servicing at least 8 players on any given sitting.
For further configuration details, you can then select how big of a disk you want for the instance. I selected the default 8GB initially. 

For the  security details, we want to ensure that the server can be accessed via SSH. We also want to make sure that Minecraft's default port is available.
Ensure port 22 (TCP) allowed as well as 25565 (TCP). Source for either can be set to value: `0.0.0.0/0`. This basically allows anyone from any IP address to contact your server

#### Elastic IP

Now we have our instance set up within EC2. However, your IP address is allocated when the instance starts. This means that the IP address may hop each time it is cycled on/off.

To bypass this, we utilize the 'Elastic IP' within EC2. You can simply allocate one to your AWS environment then assign it to the specific instance you want. 

Note: AWS does charge for all public IPv4 Addresses. This includes any IP address associated to running an instance incuding the Elastic IP

#### SSH Key

In order to access and manage your instance, we want to make sure you can generate a key pair. Either upload your own key or generate one following the instructions popup

### Linux Setup

Amazon Linux 2 AMI is built on top of Fedora. In order to get the server running we must:
- Download and install Java
- Download your specific server .jar file
- Create a specific directory for the server
- Create a specific user that has the proper permissions for the directory
- Create a service for the server that will automatically start/stop with the instance

#### Installing Server files

First lets install Java: `sudo yum install java`

Now we want to create a new directory where we store our server files:

```
sudo su
mkdir /opt/minecraft/
mkdir /opt/minecraft/server/
cd /opt/minecraft/server
```

Now lets get our server file into this directory. At this time I wanted to use a Fabric server, replace the URL with whichever server version you would like

```
wget https://meta.fabricmc.net/v2/versions/loader/1.21.4/0.16.12/1.0.1/server/jar
```

#### Dedicated User and permissions

Create a dedicated user for running the server. Don't forget to grant permission to the directory we made earlier

```
sudo adduser minecraft
sudo chown -R minecraft:minecraft /opt/minecraft/
```

#### System Service

Now lets create our service that will automatically start/stop with the server. We can do this by making a new file: `/etc/systemd/system/minecraft.service`

```
touch /etc/systemd/system/minecraft.service
nano /etc/systemd/system/minecraft.service
```

Now we want to load this minecraft.service file in order to properly start/stop the minecraft server itself

```
[Unit]
Description=Minecraft server on startup
Wants=network-online.target

[Service]
User=minecraft
Group=minecraft
WorkingDirectory=/opt/minecraft/server
ExecStart=/usr/bin/screen -DmS mc /usr/bin/java -Xmx6G <server.jar> nogui

ExecStop=/usr/bin/screen -p 0 -S mc -X eval 'stuff "save-all"\015'
ExecStop=/usr/bin/screen -p 0 -S mc -X eval 'stuff "say SERVER SHUTTING DOWN IN 10 SECONDS..."\015'
ExecStop=/usr/bin/sleep 5
ExecStop=/usr/bin/screen -p 0 -S mc -X eval 'stuff "say SERVER SHUTTING DOWN IN 5 SECONDS..."\015'
ExecStop=/usr/bin/sleep 5
ExecStop=/usr/bin/screen -p 0 -S mc -X eval 'stuff "stop"\015'

[Install]
WantedBy=multi-user.target
```

We want to use `screen` as the built-in dependancy here in order to manage the server once its running. Simply switch user to the minecraft user and use `screen -r` in order to reattach and use the cli as normal. Please view the sources section in orer to read more about the screen command

Once you're done, make sure to grant the correct permissions and restart the daemon

```
chmod 664 /etc/systemd/system/minecraft.service
systemctl daemon-reload
```

## Sources
- [How to Run a Minecraft Server on AWS For Less Than 3 USD a Month](https://sidoine.org/how-to-run-a-minecraft-server-on-aws-for-less-than-3-usd-a-month/)
- [This](https://www.youtube.com/watch?v=_1xtKGspjEA&t=386s) YouTube guide by [@WonderfulWhite](https://www.youtube.com/@WonderfulWhite)
- [AWS - Create a key pair for your Amazon EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html)
- [AWS - Connect to your Linux instance with SSH](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-to-linux-instance.html)
- [Understanding Systemd Units and Unit Files](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files)
- [Understanding Screen in Linux](https://www.geeksforgeeks.org/screen-command-in-linux-with-examples/)
