# EFS DEMO

Welcome to the EFS demo lesson where you will implement a simple Elastic File System (EFS) architecture within an AWS VPC.

## Stage 1 - APPLY BASE INFRASTRUCTURE template
In this stage, we will create a basic infrastructure in the us-east-1 region, consisting of the following components:

- A VPC using the 10.16.0.0/16 IP address range
- Two Web Subnets: WebA and WebB
- Two App Subnets: AppA and AppB
- An Internet Gateway
- Security Groups
- Instance Role
- SSM Endpoints
- Route Tables
- Two EC2 Instances, one running in WebA and the other in WebB

To apply the base CloudFormation template, please follow these steps:
1. Click [here](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?templateURL=https://techidence.s3.amazonaws.com/aws_projects/aws_efs/EFSInfra.yaml&stackName=EFS-Stack) to access the CloudFormation console.
2. Tick the "AWS::IAM::Role capabilities" checkbox.
3. Click on "Create Stack".
4. Wait until the "EFS-Stack" reaches the "CREATE_COMPLETE" status before proceeding to the next stage.

![Untitled](/images/Untitled.png)

## Stage 2 - Create EFS File System and Mount Points
In this stage, we will create an EFS file system and configure mount points. Follow the steps below:

1. Go to the [EFS Console](https://console.aws.amazon.com/efs/home?region=us-east-1).
2. Click on "Create file system".
3. Set the name to "My-EFS".
4. Choose the VPC "My-vpc" from the dropdown menu.
5. Click on "Customize" to configure advanced options.
	![Untitled](/images/Untitled1.png)
6. Ensure the following options are set as default:
   - Enable Automatic Backups: Enabled (integrates with AWS Backups for automatic backups)
   - Lifecycle Management: 30 days since last access (ensures cost-effective storage by moving files to lower-cost tiers)
   - Performance Mode: General Purpose (default and suitable for most use cases with low latency and good overall performance). MAX I/O provides higher latency but better performance for large-scale parallel workloads.
   - Throughput Mode: Bursting (performance scales with the size of the storage, similar to EBS). Provisioned allows independent control of performance.
   - Encryption: Enable encryption of data at rest (uses KMS to encrypt all data at rest). Optionally, you can select a specific encryption key.
   ![Untitled](/images/Untitled2.png)
7. Click on "Next".
8. Make sure that VPC "My-vpc" is still selected.
9. Configure mount targets, one per Availability Zone (AZ).
   - For the top row, select "us-east-1a", "sn-app-A" as the subnet, and "EFS-InstanceSG-XXXXX" as the security group (where "XXXXX" represents randomness at the end of the name).
   - For the bottom row, select "us-east-1b", "sn-app-B" as the subnet, and "EFS-InstanceSG-XXXXX" as the security group.
   ![Untitled](/images/Untitled3.png)
10. Click on "Next".
11. Skip configuring any File System Policies.
12. Click on "Next".
13. Scroll down and click on "Create" to create the file system and mount targets.
14. The file system will initially be in the "Creating" state and will transition to "Available" once ready.
	![Untitled](/images/Untitled4.png)
15. Once the file system is "Available", click on "My-EFS".
16. Go to the "Network" tab to view the Elastic Network Interfaces (ENIs) that your EC2 instances will connect to. These ENIs have IP addresses within the VPC.
17. The two mount targets will start in the "Creating" state. Wait until both are in the "Available" state before proceeding.
18. Keep clicking the "Refresh" button until both mount targets show as "Available" before continuing.
	![Untitled](/images/Untitled5.png)

## Stage 3 - Mounting EFS on InstanceA

This stage involves logging in to an EC2 instance, installing the necessary software for working with EFS, configuring and mounting the EFS file system, and performing a test.

Please follow the steps below to complete this stage:

1. Open a new tab in your web browser and navigate to the EC2 console by clicking on "Services," typing "EC2", and selecting it from the list. Alternatively, you can use the following link: [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home)

2. Click on "Running Instances" in the EC2 console.

3. Locate and select the instance named "EFS-Instance-A." Right-click on it and choose "Connect".

4. Select "Session manager" as the connection method and click on "Connect".
	![Untitled](/images/Untitled6.png)

5. Once you have successfully connected to the EC2 instance, verify that there are no currently mounted EFS file systems by running the following commands:
   ```
   sudo ls -la /
   df -k
   ```

6. To enable EFS support, which is based on NFS, you need to install a required package. Run the following command to install it:
   ```
   sudo yum -y install amazon-efs-utils
   ```

7. Create a mount point for the EFS file system by executing the following command:
   ```
   sudo mkdir -p /efs/wp-content
   ```

8. Edit the "/etc/fstab" file, which controls the file systems mounted on the Linux server, using the following command:
   ```
   sudo nano /etc/fstab
   ```

9. At the bottom of the file, paste the following line:
   ```
   file-system-id:/ /efs/wp-content efs _netdev,tls,iam 0 0
   ```

   Note: Replace "file-system-id" with the actual file system ID. To find the file system ID, return to the EFS console by clicking on the following link: [EFS Console](https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems). Locate the file system ID (e.g., "fs-xxxxxx") for the "My-EFS" file system and copy it to your clipboard. Return to the Session Manager session on the instance, position your cursor over ":", delete to the start of the line, and paste the file system ID.
	![Untitled](/images/Untitled8.png)
10. Save the changes to the "/etc/fstab" file by pressing `Ctrl+O`, then press `Enter` and exit the editor by pressing `Ctrl+X`.

11. Next, mount the EFS file system to the specified mount point by running the following command:
    ```
    sudo mount /efs/wp-content
    ```

12. Verify that the EFS file system is successfully mounted by running the command:
    ```
    df -k
    ```

13. Let's add a test file to the mounted EFS file system. Execute the following commands:
    ```
    cd /efs/wp-content
    sudo touch myefsfile.txt
    sudo nano myefsfile.txt
    ```

14. Enter some cool content in the file, in my own case, I added "I Love AWS" and save it by pressing `Ctrl+O` and press `Enter`. Then exit the editor by pressing `Ctrl+X`.

By following these steps, you will be able to mount the EFS file system on "EFS-Instance-A" and perform the necessary configurations and tests.

# Stage 4 - Mounting EFS on InstanceB

Follow these steps to mount the EFS (Elastic File System) on InstanceB:

1. Access the AWS Management Console and navigate to the EC2 service. You can use the following link: [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home)
2. Click on "Running Instances".
3. Locate and select the instance named "EFS-Instance-B".
4. Right-click on the instance and choose "Connect".
5. Select "Session Manager" and click "Connect".
	![Untitled](/images/Untitled7.png)
6.	Before proceeding, ensure that there are no currently mounted EFS file systems. Run the following commands in the session:

	```bash
	sudo ls -la /
	df -k
	```

7.	To enable EFS support, even though it is based on NFS (Network File System), you need to install the required package. Run the following command:

	```bash
	sudo yum -y install amazon-efs-utils
	```

8.	Now, let's proceed with mounting the EFS file system on this instance. First, create the mount point for the EFS file system by running:

	```bash
	sudo mkdir -p /efs/wp-content
	```

9.	Next, open the `/etc/fstab` file using the following command:

	```bash
	sudo nano /etc/fstab
	```

10.	At the bottom of the file, add the following line. Replace `file-system-id` with the actual File System ID:

	```bash
	file-system-id:/ /efs/wp-content efs _netdev,tls,iam 0 0
	```
	
11.	To find the File System ID, go back to the EFS console at [https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems](https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems). Locate the File System ID (`fs-xxxxxx`) of the A4LEFS file system and copy it to your clipboard. Return to the Session Manager session on the instance and replace `file-system-id` with the copied File System ID.
	![Untitled](/images/Untitled8.png)

12.	Save the changes to the `/etc/fstab` file by pressing `Ctrl+O`, then press `Enter` key and then exit the editor by pressing `Ctrl+X`.

13.	Next, mount the EFS file system in the mount point by running the following command:

	```bash
	sudo mount /efs/wp-content
	```

14.	To verify that the mount was successful, run the following command:

	```bash
	df -k
	```

15.	Now, navigate to the mounted folder and check if the content from the previous EC2 instance is present:

	```bash
	cd /efs/wp-content
	ls -la
	cat myefsfile.txt
	```
	
 ![Untitled](/images/Untitled9.png)

16.	You have now successfully mounted the EFS file system on InstanceB.

## Stage X - Cleanup

To perform the cleanup tasks for Stage X, please follow the steps outlined below:

1. Access the Amazon Elastic File System (EFS) console by visiting the following URL: [https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems](https://console.aws.amazon.com/efs/home?region=us-east-1#/file-systems).

2. Locate and select the `My-EFS` File System from the available options.

3. Click on the `Delete` button.

![Untitled](/images/Untitled10.png)

4. In the provided input box, either type or paste the FS-XXXXX id associated with the file system you wish to remove.

5. Click on the `Confirm` button to initiate the removal of the file system.

6. Please wait patiently for the removal process to complete.

7. Once the removal is finished, navigate back to the CloudFormation console by accessing the following URL: [https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks?filteringText=&filteringStatus=active&viewNested=true&hideStacks=false).

8. Locate and select the `EFS-Stack` stack from the list of available stacks.

![Untitled](/images/Untitled11.png)

9. Click on the `Delete` button.

10. Confirm the deletion by clicking on the `Delete Stack` button.

11. Once the stack has been successfully deleted, your AWS account will be in the same state it was at the beginning of the demo.

By following these steps, you will be able to complete the cleanup process effectively.
