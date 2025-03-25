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

- Here DevOps engineer will use lambda function and write python code which will talk to AWS APIs.
- We'll write lambda function for EBS snapshots. Python code we've written will talk to AWS API and it will get all info about EBS snapshot if they are in use or stale and any volume is actually using that snapshot. We can delete the snapshots using same lambda function.
- As lambda functions are event driven, we can trigger them using cloudwatch.

-------------------------------------------------------------------------------------------------------------------------------------

# Problem statement :- There are some EBS volume snapshots. Dev has created EC2 for which he uses volume (inbuilt). For the volume he has created multiple snapshots (Snapshot is simply copy of the image). Later dev has deleted the EC2 whereas volume and snapshots are not deleted.

Solution approach Steps :-

1. Fetch all EBS snapshots 
2. Filter out snapshots that are stale and delete

# Solution Implementation

1. Create EC2 and create snapshot from volume of that EC2. While creating EC2, in section "Configure storage" we can see volume is attached by default. So with instance comes the volume. If want to check volume inside EC2, go to “Storage” section.

<img width="562" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/7f64ac58-a4a1-4ed9-a61f-251d78266d2b">

We can also see the volume created in EC2 dashboard

<img width="536" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/448e7291-eb5d-406d-a5c1-5de8e12441f0">

If want to check volume inside EC2, go to “Storage” section

<img width="595" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/b450d34d-bf34-470e-b28e-071edb513bb6">


2. Go to EC2 Dashboard and create snapshot attahing existing volume to it

<img width="611" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/a9767ea3-bc97-4ee4-8b46-19bcb4aadcb5">

*_Now after sometime, anyone want to delete EC2, volume and snapshot. While deleting instance, volume got deleted but he forgot to delete volume. In such case he will use lambda function_*

3. Go to lambda function and create one. Go to source code and add code. Click deploy and then test. This will fail as by default lambda function as the role doesn't have permission to describe the snapshot.

<img width="628" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/df852a7b-fbc4-44d7-abfb-3db5b3a618cd">

<img width="603" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/abce9c21-702b-4a55-8ee1-12a35473cf24">

<img width="586" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/41c7ed66-6847-43c4-99f9-632cf00492af">

- Now go to configuration tab of lambda function and edit execution time to 10 sec
- Default execution time for lambda is 3 sec. Its better to keep this minimal as AWS will charge us for parameter


4. Now to add permissions for the IAM role, go to IAM policies and create custom policies.

- The custom policies need to be such that they have least privileges to any AWS service.
- Create policy and select EC2. In actions allowed select "describe snapshot" and "delete snapshot" as we want to list and delete snapshots.

<img width="606" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/aab621ee-fe44-4b0d-8df8-161001513b0f">

- Also create "describe volume" and "describe ec2" and then attach both to our role.

5. Now when we test the script, it gets executed but doesn't delete any snapshot.

<img width="571" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/a9961beb-e72c-4fff-8bc0-d23526f3d04c">

- Now we can check when was the snapshot previously used. Lets assume if snapshot is used 30 days ago, delete it. Add this condition to our code and test
- Now we can configure condition in our code that if snapshot is 30 days old or used, delete it, when we run the test it gets deleted and we get message as as volume assocaited with volume was not found.

_*This is how we can manage Cost optimization in our organization as DevOps and cloud engineer*_


<img width="607" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/48d93d44-f524-401c-99a0-8cf7a4adbcc2">


# Integrate with CloudWatch

- Lambda functions are event driven in nature, but here we were running them manually.
- Go to cloudwatch to invoke the function. Select events and rules. Create rule and schedule it to everyday to run script without manual intervention







