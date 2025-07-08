# Migrating a Server Using AWS MGN

This guide outlines the steps to migrate a server from one AWS region to another using the AWS Migration Hub (AWS MGN) service. AWS MGN simplifies the process of replicating and launching servers in a new region. Follow the steps below to achieve a successful migration:

## High-level Architecture for the Migration
![image](https://github.com/Tech-Nat/aws-cloud-migration/assets/97749491/ac75176b-cc75-4681-b1c5-9e472bfd5ce3)

## Step 1: Prepare the Source Server

1. Launch an Amazon t2.micro Linux EC2 instance in the source region.
   - Enable a public IP for the instance.
   - Open inbound ports 22 (SSH) and 80 (HTTP).
2. SSH into the Linux EC2 instance as root.
   - Install and enable the Apache HTTP server:
     ```bash
     sudo yum install httpd
     sudo systemctl enable httpd
     sudo systemctl start httpd
     ```
3. Create an `index.html` file in the `/var/www/html/` directory with sample HTML content.
4. Access the HTML file using the instance's public IP.

## Step 2: Initialize AWS MGN

1. Go to AWS MGN > Getting Started > Set up Application Migration Service.
2. Leave all options as-is and click "Create template."
   - This initializes AWS MGN, creating necessary IAM roles and security groups.

## Step 3: Generate AWS Credentials

1. In IAM, create a new user named "MGNuser."
2. Select "Programmatic access" as the access type.
3. Attach the `AWSApplicationMigrationAgentPolicy` policy.
4. Copy the user's Username, Access Key ID, and Secret Access Key.

## Step 4: Install Replication Agent on Source Server

1. SSH into the source server.
2. Download the agent installer:
   ```bash
   wget https://aws-application-migration-service-<region>.s3.<region>.amazonaws.com/latest/linux/aws-replication-installer-init
   ```
3. Install the agent:
   ```bash
   sudo python3 aws-replication-installer-init.py
   ```
4. Provide the target region where AWS MGN is initialized.
5. Enter MGNUser's Access Key ID and Secret Access Key.
6. Press Enter to replicate all disks.

## Step 5: Provision AWS MGN Replication Server

1. Monitor the source server's lifecycle progress and replication initiation steps.
2. In EC2, find the AWS MGN replication server based on your template.

_Note: This process may take time; be patient. Once the server is provisioned, initial data sync and snapshots will be created._

## Step 6: Edit Launch Settings and Template (Optional)

1. Click on the source server > Launch settings > EC2 Launch template.
2. Copy the launch template name.
3. Go to EC2 > Launch templates > select the corresponding template.
4. Modify the template if needed:
   - Go to Actions > Modify Template > make required changes > Create template version.
   - Set the new version as the default.

## Step 7: Launch the Test Instance

1. Select the source server > Test and Cutover > Launch test instance.
2. The test instance will be launched based on your template.
3. Monitor the lifecycle progress by clicking on the Job ID.
4. A new AWS MGN Service Conversion Server should be available in EC2.
5. Test your application using the test instance's public IP.
6. Once testing is complete, mark the source server as "ready for cutover.

## Step 8: Launch Cutover Instance

1. Select the source server > Test and Cutover > Launch cutover instance.
2. The test instance will be launched based on your launch template.
3. Click on the source server > Lifecycle > click on the Job ID to observe the progress.
4. Go to EC2, where a new AWS MGN Service Conversion Server should be launched.
5. Once the conversion is complete, a test instance should be available.
6. Test your application by browsing `index.html` using your test instance's public IP.
7. After testing is complete, select the source server > Test and Cutover > Finalize cutover.

## Step 9: Archive the Replicated Server

- Select the source server > Actions > Mark as archived.


