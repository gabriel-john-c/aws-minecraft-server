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

For the initial set up, I went with an Amazon Linux isntane with t2.large type. I wanted the server to have at least 8 GB of RAM since it would be servicing at least 8 players on any given sitting

## Sources
- [How to Run a Minecraft Server on AWS For Less Than 3 USD a Month](https://sidoine.org/how-to-run-a-minecraft-server-on-aws-for-less-than-3-usd-a-month/)
- [This](https://www.youtube.com/watch?v=_1xtKGspjEA&t=386s) YouTube guide by [@WonderfulWhite](https://www.youtube.com/@WonderfulWhite)
