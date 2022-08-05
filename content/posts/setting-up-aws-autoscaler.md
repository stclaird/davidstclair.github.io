---
title: "Setting up an AWS EC2 Auto-scaler and performing a simple scaling up event."
date: 2017-12-15T15:28:38+01:00
authors: ["David StClair"]
draft: false
---
AWS auto-scalers, to me at least, are the service that standout as being one of the most “Elastic-y” in the Elastic offerings from AWS. They can be configured to automatically expand and shrink on demand.  Which by most definitions is pretty elastic. 

Auto-scalers can be configured to fire up more ec2 compute instances when demand on already running instance(s) reaches a user definable threshold.  For example if you have a group of ec2 compute instances working on a queue of jobs and that queue is keeping all of the server's processor tied up consistently for say, 10 minutes the auto-scaler will detect this and launch up another ec2 instance.

The following example will show you how to test this and watch while an EC2 instance’s CPU gets busy, the auto-scaler notices this and  automatically fires-up another instance. We will use the “stress” utility to  force the server to “get busy” so we can watch for another instance to launch.

1. Create a Auto-scaling launch configuration.  This is an object that defines the EC2 instance you are wish to launch. For example, it is here that you set parameters like instance type, User data and storage.  I am going to setup a LC that will launch an t2.micro, AWS Linux and 8GB of storage.  Pretty standard stuff except I am  going to install the “stress” utility on our instance and set it to “stress” the CPU by injecting this bash code in the user data section.

```bash
#!/bin/sh
yum install -y stress
stress --cpu 1 --timeout 300
```
