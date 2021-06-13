---
layout: post
title: Shell into an Ubuntu instance on EC2 from a browser using the Systems Manager Agent
---
This post describes how to create a shell session to an Ubuntu instance on EC2 using only the browser, with no SSH client or keys.

You can use [AWS Systems Manager](https://aws.amazon.com/systems-manager/) to create an ssh-like shell session to an instance using only a browser. No local SSH client is needed, and you don't have to create SSH keys or set up open networking ports in a Security Group. 

The session interacts with the instance using the [Systems Manager Agent (SSM Agent)](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-install-ssm-agent.html). That agent is pre-installed on Ubuntu AMIs on EC2, so all you need to do is apply one IAM Policy (AmazonSSMManagedInstanceCore) to the instance.

## 1. Create an IAM Role with Systems Manager permissions
* Login to the AWS Management Console, and go to the IAM (Identity and Access Management) dashboard.
![Open the IAM dashboard]({{ site.baseurl }}/images/SelectIAM.png "AWS IAM service")

* Select "Roles" from the left-hand menu, then click the "Create Role" button.
* In the next screen, select the "AWS Service" trusted entity, if it is not already selected. Below "Choose a use case", select EC2, then click on "Next: Permissions".
![Create a new Role]({{ site.baseurl }}/images/CreateRole-1.png "Create a new Role")

* In the "Attach permissions policies" page, type `SSMManaged` into the Filter Policies to find the specific policy you need (**AmazonSSMManagedInstanceCore**). Click on the checkbox next to  it, then click "Next: Tags".
![Attach AmazonSSMManagedInstanceCore Policy to Role]({{ site.baseurl }}/images/CreateRole-2.png "Attach Policy to Role")

* On the "Add tags (optional)" screen, just click "Next: Review".
* On the Review screen, type in a Role Name of "System-manager-Role", then click "Create role". You should see the message "The role System-manager-Role has been created."
![Name this new Role System-manager-Role]({{ site.baseurl }}/images/CreateRole-4.png "Name the new Role")

## 2. Create the EC2 instance with this new Role attached
* From the left navigation bar, select Services, EC2.
* Click on the Launch instance button, and select Launch instance. 
* Select an Ubuntu Server instance. I picked "Ubuntu Server 20.04 LTS (HVM), SSD Volume Type". 
* On the "Step 2: Choose an Instance Type" screen, I chose t2.micro from the Free tier. Click "Configure instance details".
![Choose a t2.micro instance]({{ site.baseurl }}/images/LaunchInstance-2.png "choose t2.micro instance")

* On the "Step 3: Configure Instance Details" screen, scroll down to the "IAM role" drop-down choice.  It should be "None" by default.  Click on the arrow on the right of the "IAM role" drop-down.  You should see the option "System-manager-Role" listed.  Click on it to select it.  Then, click on "Next: Add Storage".
![Apply the System-manager-Role to this instance]({{ site.baseurl }}/images/LaunchInstance-3.png "Apply the System-manager-Role to this instance")

* For Storage, you can accept the defaults. Click "Next: Add Tags".
* You can also accept the defaults for Tags. click "Next: Configure Security Group".
* We're going to create a new Security Group with no network ports open at all. (You can configure any rules you want, including allowing connections on the normal SSH port of 22, but it's not required for a Systems Manager-based session.) Click on the "X" on the far right of the default SSH rule. With no rules left,  you will see a warning that you won't be able to connect to this instance.  (We'll show them!)  Click on "Review and Launch".
![Create new Security Group with no rules]({{ site.baseurl }}/images/LaunchInstance-6.png "Create Security Group with no rules")

* Verify that you've picked an Ubuntu AMI, and that the Security Group lists no rules.  Click on "Launch".
* On the key pair selection pop-up window, choose "Proceed without a key pair", and click the acknowledge checkbox.  Then, click "Launch Instances".
![No key pair needed]({{ site.baseurl }}/images/LaunchInstance-NoKey.png "No key pair needed")

## Wait for instance to launch and initialize
* Click on "View Instances." Wait a minute or two, clicking the refresh button occasionally, until the Status Check column changes from "Initializing" to "2/2".
![Instance is now ready]({{ site.baseurl }}/images/InstanceInitialized.png "Instance is now ready")

## Connect to the instance using Session Manager
* Click the checkbox to select your instance, then click the Connect button.
![Select instance and click connect]({{ site.baseurl }}/images/SelectConnect.png "Select instance and click connect")

* Choose the Session Manager option in the connection pop-up window, then click Connect.
![Choose Session Manager connection]({{ site.baseurl }}/images/SessionManagerConnect.png "Choose Session Manager connection")

* A new browser tab (or window) should open, with shell access to your instance.  
![Shell session opens in new browser window]({{ site.baseurl }}/images/ShellAccess.png "Shell session opens in new window")

## Done!
You are logged in as the Systems Manager-created user **ssm-user** with root priviliges. (You can configure Systems Manager to log in as a specific user using the ["Run As" feature in Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-preferences-run-as.html).)

You can also create and apply the necessary Role after an instance has launched, so this technique also works with existing instances.

When you're done playing around, click "Terminate" on the session window. This Terminate button only closes the shell session, it does not terminate the instance.
