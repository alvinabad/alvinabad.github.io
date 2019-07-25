---
layout: post
title: "How to run Jenkins from a folder"
description: "If your server needs to host other things besides jenkins..."
#date: 2019-07-04 00:39:00 +0800
#tags: coding
---

If you need to run a reverse proxy for your Jenkins server and use a folder, here's a good documentation:
[https://wiki.jenkins.io/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy](https://wiki.jenkins.io/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy)

The default setup of Jenkins behind a proxy points to the root of the server like this: https://mysite.com/.
But if you need your server to host other things besides Jenkins, you will need to put Jenkins in a folder like this: https://mysite.com/jenkins/,
and then other sites will be in their own folders like this: https://mysite.com/others/.

In brief, you'll need to do two things to accomplish this:

1. Set your internal Jenkins server to use a folder. 
 Your internal Jenkins must be accessible like this: http://localhost:8080/jenkins.
 Set the JENKINS_ARG parameter of your Jenkins instance to this:
 ```
 JENKINS_ARG="--prefix=/jenkins"
 ```

2. Configure your Nginx location directive:
 ```
location ^~ /jenkins/ {
     proxy_pass http://localhost:8080/jenkins/;
     ...
 ```

For details, see [Jenkins documentation](https://wiki.jenkins.io/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy).
