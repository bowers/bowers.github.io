---
layout: post
title: Shell into an Ubuntu instance on EC2 from a browser using AWS System Manager
---
This post describes how to create a shell session into an Ubuntu instance on EC2 using AWS System Manager. No SSH client needed.

You can use [AWS System Manager](https://aws.amazon.com/systems-manager/) to create an ssh-like shell session to an instance using only a browser. No local SSH client is needed, and you can configure and use these sessions using only a browser. You also don't need to open port 22 on the instance!

The session interacts with the instance using the Systems Manager Agent (SSM Agent) installed on the instance. Since that agent is pre-installed on Ubuntu AMIs on EC2, all you need to do is apply one IAM Policy (AmazonSSMManagedInstanceCore) to the instance to let you create sessions from the browser.

## Create an IAM Role with SSM permissions
* Login to the AWS Management Console, and go to the IAM (Identity and Access Management) dashboard.  [You're using IAM with your AWS accounts, right?](https://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)
![Open the IAM dashboard]({{ site.baseurl }}/images/SelectIAM.png "AWS IAM service")
* Select "Roles" from the left-hand menu.
* Click on the "Create role" button.
* In the next screen, you're asked which Trusted Entity to use. If "AWS service" is not already highlighted, click on that.  Then, under "Choose a use case", select EC2. Click on "Next: Permissions" at the bottom.
![Create a new Role]({{ site.baseurl }}/images/CreateRole-1.png "Create a new Role")
* In the "Attach permissions policies" screen, type this into the Filter Policies to find the AWS-defined policy that lets the instance interact with System Manager:
  `SSMManaged`
  
The list of policies will automatically filter, leaving just the **AmazonSSMManagedInstancecore** policy. Click on the checkbox next to  it, then click the "Next: Tags" button at the bottom.
![Attach AmazonSSMManagedInstanceCore Policy to Role]({{ site.baseurl }}/images/CreateRole-2.png "Attach Policy to Role")
* On the "Add tags (optional)" screen, just click "Next: Review".
* On the Review screen, type in a Role Name of "System-manager-Role". Then click "Create role". You should then see the message "The role System-manager-Role has been created."
![Name this new Role System-manager-Role]({{ site.baseurl }}/images/CreateRole-4.png "Name the new Role")

## Create the EC2 instance.
* From the left navigation bar, select Services, EC2.
* Click on the Launch instance button, and select Launch instance. 
* Select an Ubuntu Server instance. I picked "Ubuntu Server 20.04 LTS (HVM), SSD Volume Type". 
* On the "Step 2: Choose an Instance Type" screen, I chose t2.micro from the Free tier. Click "Configure instance details".
![Choose a t2.micro instance]({{ site.baseurl }}/images/LaunchInstance-2.png "choose t2.micro instance")
* On the "Step 3: Configure Instance Details" screen, scroll down to the "IAM role" drop-down choice.  It should be "None" by default.  Click on the arrow on the right of the "IAM role" drop-down.  You should see the option "System-manager-Role" listed.  Click on it to select it.  Then, click on "Next: Add Storage".
![Apply the System-manager-Role to this instance]({{ site.baseurl }}/images/LaunchInstance-2.png "Apply the System-manager-Role to this instance")
* You can accept the defaults here, so click on "Next: Add Tags".
* You can also accept the defaults here, so click on "Next: Configure Security Group".
* Here, we're going to create a new Security Group with NO rules. (You can configure any rules you wish; this is just to demonstrate that you don't need any ports open on the instance to shell into it.) Click on the "X" on the far right of the default rule (SSH, TCP, port 22) to delete that default rule.  There should be no rules left, and you should see a warning at the bottom that you will not be able to connect to this instance.  (We'll show them!)  Click on "Review and Launch".
![Create new Security Group with no rules]({{ site.baseurl }}/images/LaunchInstance-2.png "Create Security Group with no rules")
* Verify that you've picked an Ubuntu AMI, and that the Security Group lists no rules.  Then click on "Launch".
* On the key pair selection pop-up window, choose "Proceed without a key pair", and click the acknowledge checkbox.  Then, click "Launch Instances".
![No key pair needed]({{ site.baseurl }}/images/LaunchInstance-NoKey.png "No key pair needed")

## Wait for instance to launch and initialize
* Click on "View Instances", and wait a minute or two, clicking the refresh button occasionally, until the Status Check column changes from "Initializing" to "2/2".
![Wait for instance to initialize]({{ site.baseurl }}/images/InstanceInitializingScreenshot.png "Instance initializing")
![Instance is now ready]({{ site.baseurl }}/images/InstanceInitialized.png "Instance is now ready")

## Shell into instance with browser using Systems Manager
* From the Services menu, select "Systems Manager", which is a service listed under the "Management and Governance" list.
![Go to Systems Manager dashboard]({{ site.baseurl }}/images/SelectSystemsManager.png "Go to Systems Manager dashboard")
* Choose Fleet Manager from the left navigation bar.
![Go to Fleet Manager tool]({{ site.baseurl }}/images/FleetManager.png "Go to Fleet Manager tool")
* You should see a list of instances that are manageable by System Manager. Click on the radio button next to your instance.  Then, from the Instance Actions drop-down menu, choose "Start session".
![Go to Fleet Manager and launch session]({{ site.baseurl }}/images/FleetManager-StartSession.png "Go to Systems Manager dashboard")
* A new browser tab (or window) should open, with shell access to your instance.  
![Shell session opens in new browser window]({{ site.baseurl }}/images/ShellAccess.png "Shell session opens in new window")

## Done!
You are connected to the instance as user **ssm-user**, and you can use sudo. On my window, the little $ of the command prompt can be hard to see, but it's there.

When you're done playing around, click "Terminate" on the shell window to close the shell session.
