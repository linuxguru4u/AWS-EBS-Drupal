# AWS-EBS-Drupal
Drupal deployment into AWS Elastic Bean Stalk auto-scaling 
# AWS-EBS-Drupal
Drupal deployment into AWS Elastic Bean Stalk auto-scaling 
Build a Drupal Website on Amazon Web Services

Amazon Web Services (AWS) provides on-demand computing resources and services in the cloud, with pay-as-you-go pricing.

You can use AWS Elastic Beanstalk to deploy Drupal on AWS in a matter of minutes. Elastic Beanstalk handles the details of your hosting environment, including provisioning AWS resources such as application servers, and configuring load balancing, scaling, and monitoring.

Architecture

In this implementation, you'll use the following AWS resources:

Application servers from Amazon Elastic Compute Cloud (Amazon EC2), known as instances
Storage space from Amazon Elastic File System (Amazon EFS) that your instances can share, known as a file system
A managed relational database from Amazon Relational Database Service (Amazon RDS), known as a DB instance
A load balancer from the Elastic Load Balancing service, to distribute traffic from your users to your application servers
Scaling services from the Auto Scaling service, to ensure that you have a minimum number of available application servers, and can add or remove application servers as demand on your Drupal website changes.
The following diagram shows the architecture for this tutorial:

Hosting Drupal architecture.
Tutorial

This tutorial walks you through the process of deploying Drupal on AWS. We'll use the AWS Management Console to access AWS. Before you begin, see Prerequisites.

Steps

Prepare Storage for Your Static Assets
Create a Database
Download Drupal
Deploy Drupal
Clean Up

#Prerequisites

Complete these steps before using AWS Elastic Beanstalk to host Drupal on AWS.

#Sign Up for AWS

When you sign up for Amazon Web Services (AWS), your AWS account is automatically signed up for all services in AWS and you can start using them immediately. You are charged only for the services that you use.

If you created your AWS account less than 12 months ago, you can get started with AWS for free. For more information, see AWS Free Tier.

If you don't have an AWS account already, use the following procedure to create one.

To create an AWS account

Open https://aws.amazon.com/, and then choose Create an AWS Account.

Follow the online instructions.

Part of the sign-up procedure involves receiving a phone call and entering a PIN using the phone keypad.

Select a Region

Amazon has data centers in different areas of the world. Correspondingly, AWS products are available to use in different regions. Not every region supports every AWS resource. For this tutorial, you must select a region where Amazon Elastic File System is available. For more information, see Amazon Elastic File System Regions and Endpoints in the Amazon Web Services General Reference.

#Create a Security Group

A security group acts as a firewall that controls the traffic allowed to reach the AWS resources to which the security group is assigned. You add rules to your security group that specify the protocol, port, and source for the allowed traffic. Note that you can modify the rules for a security group at any time; the new rules take effect immediately.

For this tutorial, we'll create a security group and add the following rules:

Allow inbound SSH connections from your computer
Allow inbound NFS connections from EC2 instances associated with the group
To create and configure your security group

Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

In the navigation bar, choose the region that you decided to use to host your Drupal website.

In the navigation pane, choose Security Groups, Create Security Group.

Type drupal-sg as the name of the security group, and provide a description. Your default VPC is automatically selected.

On the Inbound tab, add rules as follows:

Choose Add Rule. For Type, choose SSH. For Source, choose Custom and type the public IP address of your local computer, in CIDR notation.

You can get your IP address using a service. For example, we provide the following service: http://checkip.amazonaws.com/. To locate another service that provides your IP address, use the search phrase "what is my IP address." If you are connecting through an Internet service provider (ISP) or from behind a firewall without a static IP address, you must specify the range of IP addresses used by client computers.

Choose Add Rule. Choose NFS for Type and Anywhere for Source.

Create a security group with rules for SSH and NFS
Choose Create.

Choose the drupal-sg security group. On the Inbound tab, choose Edit.

For the NFS rule, for Source, choose Custom. Start typing drupal-sg in the text box, and select the security group from the list.

Choose Save.

For more information, see Security Groups in the Amazon EC2 User Guide for Linux Instances.

#Create a Key Pair

AWS uses public-key cryptography to secure the login information for your instances. A Linux instance has no password; you use a key pair to log in to your instance securely. You specify the name of the key pair when you launch your instance, then provide the private key when you log in using SSH.

If you haven't created a key pair already, you can create one using the Amazon EC2 console.

To create a key pair

Open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.

In the navigation bar, choose the region that you decided to use to host your Drupal website.

In the navigation pane, under NETWORK & SECURITY, choose Key Pairs.

Choose Create Key Pair.

Type a name for the key pair in Key pair name and choose Create.

The private key file is automatically downloaded by your browser. The base file name is the name you specified as the name of your key pair, and the file name extension is .pem. Save the private key file in a safe place.

Important
This is the only chance for you to save the private key file. You'll need to provide the name of your key pair later in this tutorial, and the corresponding private key each time you connect to the instance.
If you will use an SSH client on a Mac or Linux computer to connect to your Linux instance, use the following command to set the permissions of your private key file so that only you can read it:

$ chmod 400 your_user_name-key-pair-region_name.pem

#Step 1: Prepare Storage for Your Static Assets

You can store the static assets for your Drupal website, such as images and videos, in Amazon EFS. Amazon EFS enables you to create a file system that multiple EC2 instances can mount and access at the same time.

To create a file system

Open the Amazon Elastic File System console at https://console.aws.amazon.com/efs/.

Choose Create file system.

On the Configure file system access page, do the following:

For VPC, choose your default VPC.

For Create mount targets, select all the Availability Zones.

For each Availability Zone, ensure that the value for Security group is the security group that you created in Prerequisites (for example, drupal-sg).

Choose Next Step.

On the Configure optional settings page, do the following:

For Add tags, in Value (for the Name tag), type a name for the file system.

For Choose performance mode, keep the default option, General Purpose.

Choose Next Step.

On the Review and create page, choose Create File System.

After the file system is created, copy the file system ID and save it for future use in this tutorial.

#Step 2: Create a Database

Drupal uses a database to store information. The most common database for Drupal is MySQL.

Amazon RDS provides managed relational databases. The basic building block is a DB instance, which is an isolated database environment in the AWS Cloud. A DB instance can contain multiple databases.

In this step, you launch a Multi-AZ DB instance. In a Multi-AZ deployment, Amazon RDS automatically provisions and maintains a synchronous standby replica in a different Availability Zone. The primary DB instance is synchronously replicated across Availability Zones to a standby replica to provide data redundancy, eliminate I/O freezes, and minimize latency spikes during system backups. In the event of a planned or unplanned outage of your DB instance, Amazon RDS automatically switches to the standby replica.

To launch a DB instance

Open the Amazon RDS console at https://console.aws.amazon.com/rds/.

On the navigation bar, select the same region that you used to create your EFS file system.

On the navigation pane, choose Instances.

Choose Launch DB Instance.

On the Select Engine page, on the MySQL tab, choose Select.

On the Do you plan to use this database for production purposes page, under Production, choose MySQL, and then choose Next Step.

On the Specify DB Details page, do the following:

(Optional) For DB Instance Class, select an instance size that is large enough to meet the demand of your production workload. We recommend that you start with db.m3.medium and perform load testing to determine whether it meets your needs.

For Multi-AZ Deployment, we recommend that you select Yes.

For DB Instance Identifier, type a unique name for the instance (for example, drupal-db).

For Master Username, type a login for the master user (for example, my_db_user).

Type a password for the master user in Master Password and in Confirm Password. Record your password in a safe place.

Choose Next Step.

On the Configure Advanced Settings page, do the following:

Keep the default virtual private cloud (VPC) and default subnet group selected.

For Publicly Accessible, choose No.

For Database Name, type a name for the initial database (for example, my_drupal_database).

For Enable Enhanced Monitoring, choose No.

Choose Launch DB Instance.

After a few minutes, the launch completes. When you see the message that your instance is being created, choose View Your DB Instances.

Initially, the status of your DB instance is creating. After the status changes to available, your DB instance is ready for use.

On the Details tab, under Security and Network, open the link next to Security Groups to view the security group in the Amazon EC2 console in a new browser window. Do the following to allow resources associated with your security group to receive traffic from the database on the database port:

On the Inbound tab, choose Edit. By default, there is a rule based on the DB engine that you chose. Choose Add Rule.

For Type, choose the same type as the existing rule. For Source, choose Custom and start typing the name of the security group that you created in Prerequisites (for example, drupal-sg). Select your security group from the list.

Choose Save.

Close the browser tab with the Amazon EC2 console. You should return to the browser window with the Amazon RDS console.

On the Details tab, under Security and Network, copy the DNS name from Writer Endpoint and save it for future use in this tutorial.

#Step 3: Download Drupal

To prepare to deploy Drupal using AWS Elastic Beanstalk, you must copy the Drupal files to your computer and provide some configuration information. AWS Elastic Beanstalk requires a source bundle, in the format of a ZIP or WAR file.

Note
This tutorial was developed and tested using Drupal 8.2.1. It will probably work with other versions of Drupal 8 as well.
To download Drupal and create a source bundle

Open https://www.drupal.org/project/drupal/releases/ in a browser.

Download the latest version of Drupal 8 that is recommended for production websites.

Extract the contents of the downloaded archive to a folder and name that folder drupal.

Create a folder named .ebextensions in the same folder as the drupal folder.

Tip
On Windows, you can't use Windows Explorer to create a folder with a name that begins with a period unless you also end the folder name with a period. After you enter the name (.ebextensions.), the final period is removed automatically.
Using a text editor, create a file named 99-delayed.config in the .ebextensions folder, and then do the following.

Add the following content to the file.

Replace filesystem-id with the ID of the file system that you saved earlier (for example, fs-12345678).

Replace region with the region for your file system ( for example, us-west-2).

(Optional) The script mounts the file system on the /efs directory. To change the name of this directory, update the EFS_MOUNT_DIR variable.

option_settings:
  aws:elasticbeanstalk:application:environment:
    EFS_REGION: 'region'
    EFS_VOLUME_ID: 'filesystem-id'
    EFS_MOUNT_DIR: '/efs'

files:
  "/opt/elasticbeanstalk/hooks/appdeploy/post/99_delayed_job.sh":
    mode: "000755"
    owner: root
    group: root
    content: |
      #!/usr/bin/env bash

      # Load the environment data
      EB_APP_CURRENT_DIR=$(/opt/elasticbeanstalk/bin/get-config container -k app_deploy_dir)
      EFS_REGION=$(/opt/elasticbeanstalk/bin/get-config environment -k EFS_REGION)
      EFS_VOLUME_ID=$(/opt/elasticbeanstalk/bin/get-config environment -k EFS_VOLUME_ID)
      EFS_MOUNT_DIR=$(/opt/elasticbeanstalk/bin/get-config environment -k EFS_MOUNT_DIR)

      # Mount the EFS file system and create a site directory for Drupal
      cd ${EB_APP_CURRENT_DIR}
      mkdir ${EFS_MOUNT_DIR}
      mount -t nfs4 -o nfsvers=4.1 $(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone).${EFS_VOLUME_ID}.efs.${EFS_REGION}.amazonaws.com:/ ${EFS_MOUNT_DIR}
      mkdir ${EFS_MOUNT_DIR}/sites
      
      # Move drupal/sites to the EFS mount to support automatic scaling
      mv drupal/sites drupal/sites.bak
      ln -sf ${EFS_MOUNT_DIR}/sites drupal/sites 
      [ -f ${EFS_MOUNT_DIR}/firstinit.txt ] && echo "### FirstInit Found. Skipping EFS Init... " || cp -avr drupal/sites.bak/* drupal/sites
      rm -rf drupal/sites.bak

      # Set permissions and create firstinit.txt to prevent Elastic Beanstalk from overwriting drupal/sites on subsequent deployments
      chown webapp:webapp ${EFS_MOUNT_DIR}/sites -R
      [ -f ${EFS_MOUNT_DIR}/firstinit.txt ] && echo "### FirstInit Found" || echo "First Init Complete" > ${EFS_MOUNT_DIR}/firstinit.txt
Using a text editor, create a file named prod.config in the .ebextensions folder, and add the following content:

option_settings:
  - namespace: aws:elasticbeanstalk:container:php:phpini
    option_name: document_root
    value: /drupal
  - namespace: 'aws:autoscaling:asg'
    option_name: MinSize
    value: 2
  - namespace: 'aws:autoscaling:asg'
    option_name: MaxSize
    value: 4
Create a ZIP file from the drupal and .ebextensions folders, using one of the following steps, depending on your operating system:

Windows — In Windows Explorer, select both the drupal and .ebextensions folders, right-click, and then choose Send to, Compressed (zipped) folder. Name the file drupal.zip.

Mac OS X and Linux — Use the following command to create drupal.zip:

zip -r -X ./drupal.zip .

#Step 4: Deploy Drupal

AWS Elastic Beanstalk makes it easy to deploy, manage, and scale your Drupal site on AWS.

Prerequisites

Create a security group as described in Create a Security Group.
Create a key pair as described in Create a Key Pair. You must specify a key pair when you configure application servers or you can't connect to them.
Create a DB instance using Amazon RDS as described in Step 2: Create a Database.
Create a ZIP file for Drupal as described in Step 3: Download Drupal. You must specify this file when you launch the application server.
Launch the Application Server Using Elastic Beanstalk

Provide information about your application version to Elastic Beanstalk. Elastic Beanstalk provisions the AWS resources that you need to run Drupal, such as EC2 instances.

Important
If the New Environment wizard does not show the screens described in the following procedure, you are using a preview of the New Environment wizard. The directions to use the preview wizard are in the procedure following this one.
To launch Drupal using Elastic Beanstalk

Open the Elastic Beanstalk console at https://console.aws.amazon.com/elasticbeanstalk/.

On the navigation bar, select the same region that you used to create your EFS file system.

Choose Create New Application.

Type a name and description for your application and choose Next.

On the New Environment page, choose Create web server.

For Predefined configuration, choose PHP. For Environment type, keep Load balancing, auto scaling. Using load balancing and automatic scaling enables incoming web traffic for your website to be routed across a fleet of virtual servers that can scale automatically as demand on your website changes. Choose Next.

Upload your drupal.zip file as the source of the initial application version and choose Next.

Type a name and URL for your environment. Choose Check availability to verify that this name is available and choose Next.

On the Additional Resources page, choose Create this environment inside a VPC and choose Next.

Warning
For a production Drupal website, we recommend that you do not create the DB instance using Elastic Beanstalk. If Elastic Beanstalk creates the DB instance, it will delete the DB instance when you terminate the environment and you will lose your data.
On the Configuration details page, do the following:

For Instance type, choose t2.micro.

For EC2 key pair, select your key pair.

For Application health check URL, type /robots.txt to configure the load balancer to send health check requests to that file in the application root directory using HTTP on port 80.

Choose Next.

(Optional) On the Environment Tags page, you can add a tag or continue to the next step. To add a tag, specify its key and value. When you are finished, choose Next.

On the VPC Configuration page, do the following:

For VPC, choose your default VPC.

Select ELB and EC2 for all Availability Zones.

For VPC security group, select the security group that you created in Prerequisites (for example, drupal-sg).

Choose Next.

On the Permissions page, choose Next.

On the Review page, choose Launch.

As Elastic Beanstalk creates your AWS resources, it adds information about them under Recent Events. Verify that the environment was successfully launched. If there are warnings or errors, fix the reported issues. After the launch succeeds, continue to the next task.

Use the following procedure if you are using the preview wizard.

To launch Drupal using Elastic Beanstalk (preview wizard)

Open the Elastic Beanstalk console at https://console.aws.amazon.com/elasticbeanstalk/.

On the navigation bar, select the same region that you used to create your EFS file system.

Choose Create New Application.

Type a name and description for your application and then choose Create.

From the Actions menu, choose Create New Environment.

Choose Web server environment and then choose Select.

On the Create a new environment page, do the following:

For Platform, select PHP.

For Application code, choose Upload your code and then choose Upload.

Choose Browse, select the drupal.zip file that you created as the source for your site, and then choose Upload.

Choose Configure more options.

Specify configuration settings as follows:

For Configuration presets, choose High availability.

Under Environment settings, choose Modify to open the Environment settings panel. For Name, type a name for your environment (for example, myDrupalSite). Choose Save.

Open the Instances panel. For Instance type, choose t2.micro. Choose Save.

Open the Capacity panel. For Instances, increase Min to 2 and leave Max as 4. Choose Save.

Open the Security panel. For EC2 key pair, select your key pair. Choose Save.

Open the Monitoring panel. For Health check path, type /robots.txt. Choose Save.

Open the Network panel. For VPC, choose your default VPC. Select all the subnets under Load balancer subnets and under Instance subnets, which improves availability of your Drupal site. For Instance security groups, select your security group. Choose Save.

Choose Create environment.

As Elastic Beanstalk creates your AWS resources, it adds information to the status pane. Verify that the environment was successfully launched. If there are warnings or errors, fix the reported issues. After the launch succeeds, continue to the next task.

Troubleshooting Environment Issues

The load balancer sends a health check request over HTTP to the file specified in the application health check URL. If your instances do not pass the health checks, check for the following possible causes:

The /var/www/html/drupal/robots.txt file does not exist on the instance. If this file does not exist, verify that the ZIP file you uploaded has the directory structure described in Step 3: Download Drupal.
The security groups associated with the instances do not allow HTTP traffic. By default, Elastic Beanstalk creates a security group for your load balancer (the description of the security group is "Load Balancer Security Group") that allows inbound HTTP access from anywhere, and a security group for your instances (the description of the security group is "VPC Security Group") that allows inbound HTTP access from your load balancer so that health check traffic is allowed.
For more information about this issue and other common issues with AWS Elastic Beanstalk, see Troubleshooting in the AWS Elastic Beanstalk Developer Guide.

Check the Mounted File System

The EFS file system is mounted on both instances launched by Elastic Beanstalk. You can connect to your instance and verify that the file system is mounted to the directory that you specified in the 99-delayed.config file (for example, /efs).

Note
The 99_delayed_job.sh script creates the directory that you specified even if the file system is not mounted. Therefore, you test whether the file system is mounted, not whether the directory exists.
To verify that the file system is mounted

Connect to your instances. For more information, see Connect to Your Linux Instance in the Amazon EC2 User Guide for Linux Instances.

From the terminal window for each instance, run the df -T command to verify that the EFS file system is mounted.

$ df -T
Filesystem     Type              1K-blocks    Used          Available Use% Mounted on
/dev/xvda1     ext4                8123812 1949800            6073764  25% /
devtmpfs       devtmpfs            4078468      56            4078412   1% /dev
tmpfs          tmpfs               4089312       0            4089312   0% /dev/shm
efs-dns        nfs4       9007199254740992       0   9007199254740992   0% /efs
Note that the name of the file system, shown in the example output as efs-dns, has the following form:

availability-zone.filesystem-id.efs.region.amazonaws.com:/
(Optional) Create a file in the file system from one instance, and then verify that you can view the file from the other instance.

From the first instance, run the following commands to create the file:

$ cd /efs
$ sudo touch test-file.txt
From the second instance, run the following command to view the file:

$ ls /efs
firstinit.txt  sites  test-file.txt
Troubleshooting File System Mounting Issues

If the volume is not mounted, check for the following possible causes:

The security group associated with the file system does not allow NFS traffic from the instances. For more information, see Create a Security Group.
The file system that you specified in 99-delayed.config does not exist. Be sure that you replaced filesystem-id and region with the correct values for the file system.
For more information about the cause of the issue, see the /var/log/eb-activity.log file on the instance.

Install Drupal

After your application server is launched, you can install Drupal.

To install Drupal

Choose the URL of the environment (for example, mydrupalsite.us-west-2.elasticbeanstalk.com). If everything is working, the browser displays the Drupal installation page.

On the Choose language page, select a language and choose Save and continue.

The Drupal Choose language page
On the Choose profile page, choose Standard, Save and continue.

The Drupal Choose profile page
If you see the Verify requirements page, address any errors and then continue.

On the Database configuration page, provide the information for your DB instance.

The Drupal Database configuration page
After the configuration completes, the Configure site page is displayed. Provide the requested information and choose Save and continue.

After the installation completes, the Administration page is displayed. You can set up and start using your Drupal site.

#Step 5: Clean Up

Now that you've completed step 4, your Drupal website is deployed and ready for a production workload.

If you are finished with your Drupal website, you should clean up the AWS resources that you created for this tutorial to prevent your account from accruing additional charges.

Delete the Elastic Beanstalk Resources

When you terminate your environment, Elastic Beanstalk cleans up the resources that it created. For example, it deletes the load balancer and Auto Scaling group and terminates the EC2 instances.

To delete the Elastic Beanstalk resources

Open the Elastic Beanstalk console at https://console.aws.amazon.com/elasticbeanstalk/.

Delete the application versions as follows:

Choose the name of your application, and then choose Application Versions.

Select your application versions, and then choose Delete.

When prompted for confirmation, choose Delete.

Choose Done.

Terminate the environment as follows:

Choose Environments and then select your environment.

Choose Actions, Terminate Environment.

When prompted for confirmation, choose Terminate.

After termination is complete, choose Actions, Delete Application.

Delete the Amazon RDS Database

To delete the database

Open the Amazon RDS console at https://console.aws.amazon.com/rds/.

In the navigation pane, choose Instances.

Select the DB instance.

For Instance Actions, choose Delete.

When prompted, select No for Create final Snapshot, select the confirmation check box, and choose Delete.

Delete the Amazon EFS File System

To delete the file system

Open the Amazon Elastic File System console at https://console.aws.amazon.com/efs/.

Select the file system.

For Actions, choose Delete file system.

When prompted, type the ID of the file system and then choose Delete File System.

