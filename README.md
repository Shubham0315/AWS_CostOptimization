# AWS_CostOptimization

- Most devops engineers use Cost Optimization on day to day basis so as to reduce overhead of infrastructure and optimize infrastructure cost which are primary goals to move into cloud.
- This is for Startup or midscale organizations as they don't need dedicated person to monitor.
- As a devops engineer in startup, instead of building on premises data centers, we can onboard towards cloud platform. So instead of building data centres and everything, we can move to cloud. But this is good as long as we manage the resources efficiently on cloud as cost aspect comes in picture. Setting up infra by own, paying resources for the work, is big challenge for startups. So easy go to solution for them is public cloud.

  - e.g:-  Developer has IAM access and he created EC2 instance and attached volume to it (without volume we cannot store data inside EC2).
  - Volume has sensitive info so organization need backup of volume. So developer takes snapshots of volume on daily basis (snapshot means backup in AWS).
  - After all work, Dev deletes EC2 as their work is completed but doesn't delete volume attached. So the backup snapshots taken are also not deleted. So AWS keeps charging us for all snapshots and volumes.

  - e.g:- Dev has created S3 bucket but they are not using the same. He forgets to delete it. AWS will keep charging for the same as long as it is in active state.


- There are almost around 200+ resources on AWS. There might be "**STALE Resources**" (someone created but forgot to delete). So cloud cost will go higher than expected as we're not managing cloud cost efficiently.
- The primary role of Devops engineer is to make sure cloud cost should go down by looking at stale resources on platform.
- DevOps Engineer can do things like below if any stale resources found :-
  - Send out notifications to concerned team about stale resources
  - Delete them on own if admin access is present
 
-------------------------------------------------------------------------------------------------------------------------------------

# How devops engineers can implement cost optimization by deleting stale resources?

- Here DevOps engineer will use lambda function and write python code using BOTO3 module which will talk to AWS APIs.
- We need to write these lambda functions for individual resources.
- Here, we'll write lambda function for EBS snapshots. Python code we've written will talk to AWS API and it will get all info about EBS snapshot if they are in use or stale and any volume is actually using that snapshot or volume is deleted or attached to EC2. We can delete the snapshots using same lambda function.
- As lambda functions are event driven, we can trigger them using AWS cloudwatch.

-------------------------------------------------------------------------------------------------------------------------------------

# Problem statement :- There are some EBS volume snapshots. Dev has created EC2 for which he uses volume (inbuilt which comes with EC2). For the volume dev has created multiple snapshots (Snapshot is simply copy of the image). Later dev has deleted the EC2 whereas volume and snapshots are not deleted. Here if EC2 or volume gets deleted, snapshots are of no use.

Solution approach Steps :-

1. Fetch all EBS snapshots 
2. Filter out snapshots that are stale and delete

-------------------------------------------------------------------------------------------------------------------------------------

- What if we've intentionally created snapshots deleting EC2 and volumes. But the snapshots are just for backup purpose.
- Here also we can verify the timestamp of creation of timestamp. If the snapshot is way older, we can delete it

-------------------------------------------------------------------------------------------------------------------------------------

Solution Implementation
-
# 1. Create EC2 and attach volume (default)
-
  - Create EC2 and create snapshot from volume of that EC2.
  - While creating EC2, in section "Configure storage" we can see volume is attached by default. So with instance comes the volume. If want to check volume inside EC2, go to “Storage” section.

![image](https://github.com/user-attachments/assets/38e948dc-0684-4056-88f5-c090cf505e22)

  - We can also see the volume created in EC2 dashboard

![image](https://github.com/user-attachments/assets/92bd59b8-20d4-4e50-8d6a-2cca690deb94)
![image](https://github.com/user-attachments/assets/e526aae1-f566-430c-8af0-684cec892f8e)

  - If want to check volume inside EC2, go to “Storage” section

![image](https://github.com/user-attachments/assets/40483548-4a83-4961-b94b-0a1dc9144efb)


# 2. Create Snapshot
-
  - Go to EC2 Dashboard and create snapshot attahing existing volume to it.
  - Snapshot is image of the volume

![image](https://github.com/user-attachments/assets/87633789-4c6f-4fdf-a9e4-6d3de5bb37e6)
![image](https://github.com/user-attachments/assets/60b7c02f-478a-4a02-ac4f-da9e61baa18e)

  - Now after sometime, anyone want to delete EC2, volume and snapshot. While deleting instance, volume got deleted but he forgot to delete volume. In such case he will use lambda function
  - Here our EC2 instance has volume attached to it and it takes snapshot

# 3. Create Lambda Function
-
  - Go to lambda function and create one providing existing permissions. 

![image](https://github.com/user-attachments/assets/11eb9c6a-9ba1-4cf1-89a2-ec30bd421929)

  - Go to source code and add code.  This will fail as by default lambda function as the role doesn't have permission to describe the snapshot.
![image](https://github.com/user-attachments/assets/128619eb-b864-4fbe-9e30-7917cf380410)

  - Click deploy and then test. In test just give name and save

![image](https://github.com/user-attachments/assets/7225d40c-3042-408a-b483-c3d5a35e7a9b)
![image](https://github.com/user-attachments/assets/cfe240ef-d9a2-4a34-83e3-23f67761883f)

  - If we see in execution results, it will fail as by default lambda function execution is only of 3 seconds.

![image](https://github.com/user-attachments/assets/9598339c-e9a7-43f4-9ca7-dfd6d1d2b038)


  - Now go to configuration tab of lambda function and edit execution time to 10 sec and save.

![image](https://github.com/user-attachments/assets/baa40f13-4ccf-4700-b665-09d6ce47b897)
![image](https://github.com/user-attachments/assets/185fb80d-8c64-442a-8f3e-debe458e467d)

- Default execution time for lambda is 3 sec. Its better to keep this minimal as AWS will charge us for parameter.


# 4. Grant Permissions
-
  - For one service talking to other service we need an IAM role and required permissions.
  - To give our role a permission, click on Configurations - Permissions - Role in lambda

  ![image](https://github.com/user-attachments/assets/91338c34-3bfc-4792-8528-d3e01d7d8e84)

  - Now to add permissions for the IAM role, go to IAM policies and create custom policies.
  - Add "DescribeSnapshots" and "DeleteSnapshots" permissions to it - Attach policies

![image](https://github.com/user-attachments/assets/2cd49674-e3e2-4334-a71b-c95d4d903221)

  - Here we dont see snapshots permissions in list. So create custom policies

![image](https://github.com/user-attachments/assets/e7d4e838-fa2e-4395-bcb8-5ba0b3118b1b)

  - Go to policies - Create policy - Select service as "EC2" - Filter actions - Resources as "All" - Provide policy name - Create policy

![image](https://github.com/user-attachments/assets/dc0f81b2-db84-491a-851a-c904e3100a12)
![image](https://github.com/user-attachments/assets/a8c2899e-f1c3-46d4-913f-fe10e6e98b8b)
![image](https://github.com/user-attachments/assets/715c69d8-3296-4ead-8f68-9bca396a72c0)

  - Now go to role and attach this policy to it
  - Add permissions - Attach policy

![image](https://github.com/user-attachments/assets/01444297-232e-471c-8f87-4fdb1c516e42)
![image](https://github.com/user-attachments/assets/2b04ad9b-805a-4bb6-9b88-27d33dd39e33)

  - Now try to execute the role. Ideally it should not delete anything

  - As snapshot belong to volume which is for EC2 instances. So create "describe volume" and "describe ec2" and then attach both to our role.

![image](https://github.com/user-attachments/assets/f582c0ed-5020-4486-9a95-9260c81e4039)
![image](https://github.com/user-attachments/assets/0a0e19de-8dbe-4267-8ec5-86a1ad1bf72d)
![image](https://github.com/user-attachments/assets/2c7b9068-c3b1-4094-82d4-606be9a24081)

  - Now go to role and attach this policy

![image](https://github.com/user-attachments/assets/1b0c0465-68bb-4f1e-a2d7-3c05019269e1)
![image](https://github.com/user-attachments/assets/bbc91000-c0b5-4fc5-bf83-e7b5ea2ae54d)


# 5. Execute the Lambda Script 
-
  - Now when we test the script, it gets executed and delete any snapshot. We can check in EC2 snapshots.

![image](https://github.com/user-attachments/assets/33a021ef-7b45-4094-b973-6973950e5eb7)
![image](https://github.com/user-attachments/assets/8c7f3aaa-05ea-43df-879d-623db071d86e)

-------------------------------------------------------------------------------------------------------------------------------------

- We can also add one condition in script instead of directly deleting it
  - We can check when was the snapshot previously used.
  - Lets assume if snapshot is used 30 days ago, delete it. Add this condition to our code and test
 
- When we test script, in output we get that snapshot associated with EC2 deleted as it was not used for long time.
- Now we can configure condition in our code that if snapshot is 30 days old or used, delete it, when we run the test it gets deleted and we get message as as volume assocaited with volume was not found.

_*This is how we can manage Cost optimization in our organization as DevOps and cloud engineer*_


<img width="607" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/48d93d44-f524-401c-99a0-8cf7a4adbcc2">


# Integrate with CloudWatch

- Lambda functions are event driven in nature, but here we were running them manually.
- Go to cloudwatch to invoke the function. Select events and rules. Create rule and schedule it to everyday to run script without manual intervention







