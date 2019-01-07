# cd-demo-jd
A scale and continuous delivery demo using Jenkins on DC/OS.
- Now updated to work with the latest DC/OS (1.12), Mesosphere Github repo access and Docker credentials

## What Are We Showing
1. Manual Jenkins Configuration of an Automated CI/CD Workflow 
- Git push, then automated Jenkins Git build,  pull to Dockerhub and deployed app into DCOS.
- Shows the DCOS open source Jenkins is functional Jenkins and that DCOS can manage the automated resource handling for CICD.

![DCOS Deployed Jenkins to CICD](https://github.com/jdyver/cd-demo-jd/blob/master/img/CD-Intro.png)

2. Scaled Deployment
- Show that this is a scalable Jenkins solution by executing 50 jobs with dynamically created task executors based on the available resources from DCOS.

![DCOS - Jenkins Service Jobs](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceJenkinsOverview.png)

### Exercise 0. Prerequisites
- I just wish this wasn't so bad
#### Step 0.1 - Only needed for Manual Deployment (First run only)
##### Item 1. Clone cd-demo to your unit and create/sync it to your Github repo
- Command: `git clone https://github.com/jdyver/cd-demo-jd.git`

#### Step 0.2 - Additional Steps Needed for Scaled Deployment
##### Item 1. Setup Python3 and Pip3 (First run only needed)

 - painful 30 minute installation - Just google away to get it right # I'll probably try to change this to bash later

 a. OSX example: https://docs.python-guide.org/starting/install3/osx/

 b. Centos example: https://www.rosehosting.com/blog/how-to-install-python-3-6-4-on-centos-7/

##### Item 2. Pip3: Install requirements

    `pip3.6 install -r requirements.txt`
    
 a. CentOS (not logged in as root): Add sudo

##### Item 3. Github Create repo/branch 
- Everytime - check cd-demo folder with 'git remote -v' or 'git branch'

 a. Manually create new repo within your Github account

 b. git clone https://github.com/jdyver/cd-demo-jd.git

 c. cd cd-demo

 d. git remote set-url upstream https://github.com/<Your Username>/<New Repo Name>

 e. git push -u upstream master

 f. Skip to Step 4

 - If using from locally forked repository

      a. git checkout -b testing

      b. git push origin testing    

      c. (if push fails)
    
       - Setup Git SSH Key (https://help.github.com/articles/generating-an-ssh-key/)    
       - (ssh -T git@github.com success, but push still fails)  
            `git remote set-url origin git@github.com:jdyver/cd-demo.git` (your repo)

##### Item 4. Dockerhub:             

    - Ensure that you have a dockerhub account and you know the username and password


### Exercise 1. Edit - Manual Git build, Docker pull and Jenkins deploy
#### Step 0. Prerequisite 0.1

#### Step 1. Prepare DCOS
    
 a. Single cluster configured for CLI
 
 - If doing the 50 job (Section 2), go ahead and clear the DCOS profiles

    `dcos cluster remove --all`

 - Connect to the cluster
    
    `dcos cluster add https://<your url>`

 b. Install Marathon-LB
 
 - Get MLB IP 

 c. Install Jenkins

#### Step 2. Within your cd-demo Github repo:
    - Edit file: conf/cd-demo-app.json - line 21
        - "HAPROXY_0_VHOST": "\<Public Agent IP\>",

![Edit HAPROXY](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-Deployed-App-HAPROXY2.png)

#### Step 3. Prepare Jenkins:
- Open the Jenkins UI from DC/OS
- Credentials > System > Global Credentials > Add Credentials: Add Dockerhub account (Input Description)

![Jenkins - Credentials](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-Credentials.png)

- Select New Job:
    - Select Freestyle and give it a name
    - Source Code Mgmt Section Git:
        1. Repo URL: `https://github.com/jdyver/cd-demo` (Your Github URL for the cd-demo repo)

![Jenkins - Source Code](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-1.png)

- Setup Auto Build (polls every 1 minute for build changes)
    - Project Name > Configure > Build Triggers > Poll SCM
    - Input: `* * * * *`

![Jenkins - Polling](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-2.png)

- Build > [Add Build Step] > Docker Build and Publish > 
    - Repo Name: `jdyver/cd-demo` (Your Docker repo username + '/cd-demo')
    - Tag: `$GIT_COMMIT`
    - Registry credentials: `Dockerhub`

![Jenkins - Build](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-3.png)

-  Post-build Actions > [Add post-build action] > Marathon Deployment > [Advanced] >
    - Marathon URL: `http://leader.mesos:8080`
    - Definition File: `conf/cd-demo-app.json`
    - Docker Image: `jdyver/cd-demo:$GIT_COMMIT`

![Jenkins - Post-build](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-5.png)

- Hit save

#### Demo is now setup; To show the demo....

#### Step 4. Github Repo: edit site/_posts/2017-12-25-welcome-to-cd-demo.markdown
 - edit some text

```
jamess-mbp:cd-demo jd$ cat site/_posts/2017-12-25-welcome-to-cd-demo.markdown << EOF
---
layout: post
title:  "Welcome to James' Continuous Delivery Demonstration!"
categories: demo
---

This is an example post that you can make using Markdown to demonstrate a website being statically generated and deployed!  ...and a belated Merry Christmas!
EOF
```

#### Step 5. Once the minute poll from Jenkins completes it will see the commit and (re)deploy the Jenkins_Deployed_App

![CD Webpage Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/JenkinsSetup-4.png)

Step 6. Open a tab and go to the Marathon-LB's \<Master_IP\>

- If it doesn't open either the jenkins_deployed_app isn't completely up or the port (HAPROXY) wasn't updated so go to MLB to pull what port it is running on

![CD Webpage Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/CD-Demo-Output.png)

### Exercise 2. Deploy 50 Jobs
0. Prerequisites 0.1 and 0.2

- 0.2 Has to be done/checked every time

    `git branch`

1. Single cluster configured for CLI

     `dcos cluster remove --all`

     `dcos cluster add http://<your url>`

2. Install Jenkins

     a. Through CLI or UI

     `cd-demo jd$ dcos package install --yes jenkins`

     b. DO NOT DO - JUST FOR NOTES; OPTION (Do not do this if you do not do 2a) 

     — <b>slash at the end of the URL!</b>

     `cd-demo jd$ python3 bin/demo.py install --latest http://jdyckowsk-elasticl-108ld3uvv6r15-1683503373.us-west-2.elb.amazonaws.com/`

     — <b>slash at the end of the URL!</b>

3. Run script

— <b>slash at the end of the URL!</b>

`cd-demo jd$ python3 bin/demo.py dynamic-agents http://jdyckowsk-elasticl-108ld3uvv6r15-1683503373.us-west-2.elb.amazonaws.com/`

— <b>slash at the end of the URL!</b>

![Python Dynamic-Agents Output](https://github.com/jdyver/cd-demo-jd/blob/master/img/cd-demo_dynamic-agents.png)

##### Exercise 2 Demo
4. Go to the Jenkins UI

![Jenkins - 50 Jobs](https://github.com/jdyver/cd-demo-jd/blob/master/img/Jenkins-50-Finale.png)

What you see are 50 jobs that have been created through automation and are randomly timed to fail/succeed within 2 minutes.
- Rerun for a more mixed view of jobs that have succeeded, failed/succeeded or just always failed.

5. In the DC/OS UI, you can select the Jenkins Service and see the Jenkins executors dynamically scale out and then back in to manage these 50 jobs based on the available resources of the cluster.  In the example outputs below, there are limited resources or unlimited resources and shown by Jenkins deploying more executors automatically within DCOS based on the demand.

- Below shows Jenkins running on a single node of resources so automatically deploying 4 executors within DCOS

![Jenkins with Single Node Resources](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceJenkins1.png)

- Below shows Jenkins running on more resources so automatically deploying 8 executors within DCOS

![Jenkins with Multiple Node Resources](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceJenkins2.png)

- Once all of the jobs complete; Jenkins kills the executors and DCOS clears the resources for more / other jobs

![Jenkins Killed Executors](https://github.com/jdyver/cd-demo-jd/blob/master/img/DCOS-ServiceKilledJenkins3.png)

### TBD (In order of priority)
- demo.py: Change / remove DC/OS CLI profile check
- demo.py: Change to bash (basically a rewrite)

# --------------------------------------------------------------------
