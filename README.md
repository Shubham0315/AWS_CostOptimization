# AWS_CostOptimization

- Most devops engineers use Cost Optimization on day to day basis. This is as to reduce overhead of infrastructure and optimize infrastructure cost. This is for Startup or midscale organizations as they don't need dedicated person to monitor.
- As a devops engineer in startup, instead of building on premises data centers, move towards cloud platform. This is good as long as we manage the resources efficiently on cloud.

e.g:-  Developer has IAM access and he created EC2 instance and attached volume to it. Volume has sensitive info so organization need backup of volume. So developer takes snapshots of volume on daily basis (snapshot means backup in AWS). After all work, Dev deletes EC2 but doesn't delete volume attached. So the backup snapshots taken are also not deleted. So AWS keeps charging us for all snapshots and volumes.

e.g:- Dev has created S3 bucket but he is not using the same. He forgets to delete it. AWS will keep charging for the same as long as it is in active state.

- There are almost around 200+ resources on AWS. There might be "STALE Resources" (someone created but forgot to delete). So cloud cost will go high as we're not managing cloud cost efficiently. This is primary role of devops engineer. Devops engineer has to make sure cloud cost should go down by looking at stale resources on platform.
- DevOps Engineer can do things like below :- send out notifications to concerned team about stale resources or delete it himself if he has admin access.

# How devops engineers can implement cost optimization by deleting stale resources?

- Here DevOps engineer will use lambda function and write python code which will talk to AWS APIs.
- We'll write lambda function for EBS snapshots. Python code we've written will talk to AWS API and it will get all info about EBS snapshot if they are in use or stale and any volume is actually using that snapshot. We can delete the snapshots using same lambda function.
- As lambda functions are event driven, we can trigger them using cloudwatch.


# Problem statement :- There are some EBS volume snapshots. Dev has created EC2 for which he uses volume (inbuilt). For the volume he has created multiple snapshots (Snapshot is simply copy of the image). Later dev has deleted the EC2 whereas volume and snapshots are not deleted.

Solution approach Steps :-

1. Fetch all EBS snapshots 
2. Filter out snapshots that are stale and delete

# Solution Implementation

1. Create EC2 and create snapshot from volume of that EC2. While creating EC2, in section "Configure storage" we can see volume is attached by default. So with instance comes the volume. If want to check volume inside EC2, go to “Storage” section.

<img width="562" alt="image" src="https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/7f64ac58-a4a1-4ed9-a61f-251d78266d2b">

![image](https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/5cb7f029-f6dd-4eb5-bb83-bf8a36e4ce32)

![image](https://github.com/Shubham0315/AWS_CostOptimization/assets/105341138/dd56de45-a045-4b5a-a6b3-f063a826259b)


2. Go to EC2 Dashboard and create snapshot

