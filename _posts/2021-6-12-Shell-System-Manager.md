---
layout: post
title: Shell into an AWS Instance using System Manager
---
This post describes how to shell into an Ubuntu instance on AWS EC2 using System Manager.

You can use System Manager on AWS to shell into an instance via a browser, and without having to use the command line at all, and without having to open port 22 to anyone.
All you need is an instance with the Systems Manager Agent (SSM Agent) installed on the instance, and one Policy (AmazonSSMManagedInstanceCore) applied to the instance.
Since Ubuntu images on AWS EC2 include the Systems Manager agent, it's just a matter of a couple of clicks, and you can shell without using ssh!

## Create an IAM Role with SSM permissions##
* Login to the AWS Management Console. 
* Go to IAM.  [You're using IAM, right?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
* Select "Roles", either from the left-hand menu, or from the "IAM Resources" near the top of the center of the screen.
* Click on the "Create role" button.
* In the next screen, you're asked which trusted entity. If "AWS service" is not already highlighted, click on that.  Then, below under "Choose a use case", select EC2. Then click on "Next: Permissions" at the bottom.  [Picture: CreateRole1]
* In the "Attach permissions policies" screen, type into the Filter Policies search box:
  `SSMManaged`
* The window will automatically filter all available policies, leaving the one you want. Click on the checkbox next to "AmazonSSMManagedInstanceCore", then click the "Next: Tags" button at the bottom.
* On the "Add tags (optional)" screen, just click "Next: Review".
* On the Review screen, for Role Name enter "System-manager-Role".  Then click "Create role". You should then see the message "The role System-manager-Role has been created."

## Create the EC2 instance.##
* From the left navigation bar, select Services, EC2.
* Click on the Launch instance button, and select Launch instance.
* Select an Ubuntu Server instance, such as "Ubuntu Server 20.04 LTS (HVM), SSD Volume Type". 
* On the "Step 2: Choose an Instance Type" screen, choose t2.micro, which is in the Free tier. Click "Configure instance details".
* On the "Step 3: Configure Instance Details" screen, scroll down to the "IAM role" drop-down choice.  It should be "None" by default.  Click on the arrow on the right of the "IAM role" drop-down.  You should see the option "System-manager-Role" listed.  Click on it to select it.  Then, click on "Next: Add Storage".
* You can accept the defaults here, so click on "Next: Add Tags".
* You can also accept the defaults here, so click on "Next: Configure Security Group".
* Here, we're going to create a new Security Group with NO rules.  Click on the "X" on the far right of the default rule (SSH, TCP, port 22) to delete that default rule.  There should be no rules left, and you should see a warning at the bottom that you will not be able to connect to this instance.  (We'll show them!)  Click on "Review and Launch".
* Verify that you've picked an Ubuntu AMI, and that the Security Group lists no rules.  Then click on "Launch".
* On the key pair selection pop-up window, choose "Proceed without a key pair", and click the acknowledge checkbox.  Then, click "Launch Instances".

## Shell into instance with browser using Systems Manager##
* Now wait for the instance to launch. Click on "View Instances", and wait a minute or two, clicking the refresh button occasionally, until the Status Check column changes from "Initializing" to "2/2".
* Now time to shell into the new instance.  From the Services menu, select "Systems Manager", which is a service listed under the "Management and Governance" list.
* You should see a list of instances that are manageable by System Manager. Click on the radio button next to your instance.  Then, from the Instance Actions drop-down menu, choose "Start session".
* A new browser tab (or window) should open, with shell access to your instance.  

On my window, the little $ of the command prompt can be hard to see, but it's there)
Note that you are connected to the instance as user "ssm-user", and you have sudo privileges.

When you're done playing around, click "Terminate" on the shell window.
