Launching an EC2 Instance
This is just a simple lab where:

Fire up an ondemand instance

Allow SSH inbound via SGs

SSH onto it via the public IP

Below is how the instance looks in the AWS console


 

EC2 Change instance type
There are times that you may wish to change an instance type.  For example a real world scenario could be that you were running out memory and you spot that via Cloudwatch’s Custom Metrics that you started pushing.  In that case you decide to change instance type from t3.micro to t3.small

You are only able to change the instance type for EBS backed instances

Before you start the instance must be powered off and check what you are moving from




[ec2-user@ip-172-31-82-241 ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:            983          60         543           0         379         785
Swap:             0           0           0
With the instance powered off you can now change the instance type


Once its powered off you can make the switch and change your work (notice from the snippet how the Public IP changed as we powered off the instance and powered it back on)




[ec2-user@ip-172-31-82-241 ~]$ free -m
              total        used        free      shared  buff/cache   available
Mem:           3942          66        3726           0         149        3686
Swap:             0           0           0
 

Placement Groups
When we start an EC2 instance on AWS we don’t have much visibility of the underlying HW however we have a certain level of control via Placement Groups.  In total there are 3 strategies for Placement Groups.

Cluster - instances are placed within the same rack in the same AZ.  This allows for very low latency and high levels of throughput.  The biggest drawback for this is that if the rack fails then all the instances will suffer an outage.  Use case for this approach:

Big Data job that needs to complete as quickly as possible


Spread - This allows for instances to be placed across difference racks in a region.  The maximun number of instances per Spread group is 7 per AZ.  Great for apps that are sensitive to node failures and need to keep them as isolated as possible


Partition - This allows for Partitions to be created out of physical racks within an AZ.  You can then position the instances across the Partitions.  You can only have 7 Partitiond per AZ however these can scale to 100s of instances.  Instances can access their own metadata which will contain the Partition Group they are in.  Physical racks cannot be shared across 2 partitions.  Use cases for this approach:

HDFS

HBase

Kafka

Cassandra


The way you create a Placement Groups is via the AWS console in EC2:



Once the Placement Group has been created you can then spin up and instance and place it in there


 

EC2 Shutdown Behaviour and Termination
When you shutdown an instance from within the OS the default state for it to transition to is Stopped, this can be changed to the Instance will be Terminated.  

You can enable Termination protection which prevents an instance to be terminated via the AWS console and the AWS CLI

This shows how you can enable for a new instance


This shows how you can change the behaviour for a running instance


The key point here is if you have the following set, and instance is shutdown via the OS the result will be that the instance will transition from Running to Terminated:

Shutdown behaviour set to Terminate

Terminate Protection set to Enabled

You can see this in action here:





[ec2-user@ip-172-31-82-241 ~]$ 
Broadcast message from root@ip-172-31-82-241.ec2.internal (Mon 2020-05-18 07:03:45 UTC):
The system is going down for power-off at Mon 2020-05-18 07:04:45 UTC!

Termination Protection will only prevent from accidental termination via the AWS Console and the AWS CLI.  If the default behaviour is set to Terminate and you initiate a shutdown via the OS the instance will be Terminated

 

EC2 Launch Troubleshooting
 In the AWS Console within EC2 you can see what the limits are for your account


InstanceLimitExceeded - This is the error you will get when you try and stand up and instance and you have reached your limits.  This can be rectified by placing a call to AWS support.  At the moment the limit is set to 32 vCPUs however previously it was set to 20 instances

InsufficentInstanceCapacity - This is a limit that has been hit at the AWS end.  It means that currently there is not enough compute resource to satisfy your request in the AZ you have requested.  You can try and solve this by:

Break down the request (from 5 to 1 for example)

Try again later

Deploy a different instance type and then change later

If you are trying to spin up and instance and the instance state goes from Pending to Terminated right away it could be one of the following:

Corrupted EBS volume snapshot from which you are trying to deploy the instance from

You have reached your EBS volume limit

The Instance store backed AMI you are trying to boot from is corrupted

The EBS volume is encrypted and you dont have the KMS key to un-encrypt

 

EC2 SSH troubleshooting
Fairly simple part this one.  If you can SSH onto the Public IP of an EC2 instance it could be one of the following:

Unprotected File Error - the wrong permissions are set on the .pem key

Host Key Not Found - the wrong username has been used to SSH

Timeout - the SGs are blocking port 22 inbound traffic

 

EC2 Instance Launch Types
Ondemand - This what we mostly use when running our labs.  Great for short spikes and for apps that we don’t quite know how their workload will look like.  Billed per second after the first minute they have been up and running

Reserved - Think long term usage like DBs.  Once you know the profile of you application you can think of reserving the instance.  1yr to 3yr engament which you pay for upfront.  75% cheaper than Ondemand

Convertible Reserved - These are still reserved instances that you can change once they have been spun up.  For example the workload profile changes and you wish to switch from a Memory intensive instance type to a CPU intensive instance type.  54% cheaper than Ondemand

Scheduled Reserved instances - you can reserve instances for pre-defined window

Spot instances - you set a maximum price you are willing to pay for the instance via a spot request.  providing the spot price is below that you will get an instance providing there is capacity.  Ondemand will trump Spot requests to your Spot instance could be reclaimed with a 2 min warning

Dedicated Hosts - Allows you to have visibility of the underlying compute CPU.  The compute host completely allocated to you on a 3yr engagement.  Used for a BYOL or for businesses that have strong compliance

Dedicated Instances - You don’t get visibilty of the underlying CPU.  Your instances will still not be shared with other AWS accounts however instances from within your account can be spun up on the compute host.

 

EC2 Spot Instances Deep Dive
Offer great discounts - up to 90% off ondemand prices

The way it works is that you create a spot request with the maximum price you are willing to pay for the instance.  Providing the spot price is below is below what you are willing to pay you get to keep the instance.  if the price goes above that you lose your instance with a 2 min warning to close your work.  Once the spot price returns below your maximum your spot instance request can be satisfied again

Spot instances also allow you to set Spot Blocks.  These are blocks that protect your instances from being reclaimed from between 1-6hrs.  Instances can still technically be reclaimed during that period however it is very rare

Spot requests can be one of the following:

One-time - once the request has been fulfilled the instance will be shutdown and that will be it

Persistant - the spot request will continue to spin up instances until the request it valid for

In order to stop Persistant Spot requests from firing up instances you need to first cancel the Spot request.  In order for this to happen the Spot request needs to be in the following state:

Open

Active

Disabled

Cancelling a Spot request will not automatically stop the instances started by it. It is your responsibility to do so

Here is a diagram that shows how to terminate a Spot instance:


Lastly with Spot instances you can have Spot Fleets which allow you to have a mixture of Spot instances and possibly On-demand instances to satisfy your requirements.  The way the work is that you define your requirements (CPU, RAM, Max Price etc) and the Spot fleet will try to satisfy these.  You define a bunch of pools and the Spot fleets will fire off instances from pools in order to reach the capacity you require

Spot instances strategies include:

Lowest Price - pool with lowest price

Diversified - Distributed across pools

Capacity Optimised - Pool with optimal capcity 

EC2 Launch Types Hands on
Ondemand instances is what we use mostly when doing these labs so we can skip these

Reserved and Convertible Reserved instances can be purchased like so


Scheduled Reserved Instances can be purchased like so


Spot instances can be purchased like so - this also shows the Spot Block



 

Dedicated hosts and Dedicated instances



 

EC2 Instances Deep Dive
Think FIGHTDRMCPIXZ and this also a useful way to remember the main instance types:

R - Think Lots of RAM (In memcache)

C - Think Lots of CPU (DBs)

M - Think Medium (Apps that are balanced)

I - Think Lots of IO (DBs)

G - Think GPU (Apps that need Graphics)

T 2/3 - Think Turbo (Burstable CPU up to capacity)

Tu 2/3 - Think Turbo (Burstable CPU unlimited)

For T 2/3 once you start bursting into CPU you burn credits (you can view this on the Cloudwatch default metrics)

For Tu 2/3 you only pay for what you burst

 

EC2 AMI
AWS comes with a bunch of AMIs ready for you to use, you can also buy or rent AMIs as well as create you own ones

AMIs are allocated on a per region basis, if you wish to use an AMI in another region you will need to copy it over

AMIs are stored in S3 however you don’t get to see the bucket in which they are stored in

As a quick example I have spun up an instance, installed/started/enabled httpd.  Once that is all in place I then created an AMI from it


Once we have a custom AMI built from our instance we can spin up an instance from it - note the region as the AMI is only available where we created it.  Also note the owner which is our AWS account number



Once the instance spins up we can access the noddy web page we created without having to configure it all over again

You can copy the AMI to another region


You can also share the AMI to another AWS 


Once shared you can access the account you shared the AMI with and you will be able to see it in there.  It will be in the same region the AMI came from


However if you try and copy it you get the following

The above is the case as we didn’t grant permissions to create an EBS snapshot from the volume.  if we enabled it and reshared


When you share an AMI with another account you still remain the owner of that AMI however if they copy the AMI to another region they become the owner.  

If you dont have the above set, you can just simply start an instance using the AMI in the secondary account and make an AMI from the running instance.  For example below I have fired up and instance with the root volume encrypted with the built in KMS key.  Once the instance is up and running and I try to build an AMI from it the AMI can only be created as encrypted, I don’t get any other options


I then shared it with the secondary account - when I tried to allow “create volume permissions” from the snapshot I get the following error


I also tried to share without allowing “create volume permissions” however that failed too with the following


If you use an AMI that has an associated billingProduct such as a windows AMI, you won’t be able to share the AMI directly.  Instead you can create an AMI from an instance in the source account.  You are then able to share that AMI and grant the target account “create volume permissions” (if you dont allow permissions you wont be able to copy the AMI to another region from the target account, instead you need to spin up an instance in the region the AMI sits in originally, create an AMI and then copy that over)

Elastic IP
As we have seen previously when you stop and start an instance the Public IP can change

You can reserve an Elastic IP and attach it to an instance when it starts.  If you then need to shutdown that instance you can transfer it to another instance

You only pay for an Elastic IP when it isnt being used, so it is reserved but not attached to an instance

Really you should avoid using Elastic IPs and either use a DNS entry from Route53 or place the instances under an ELB

You can have max 5 Elastic IPs reserved.  This can be increased by placing a call to AWS 

Before you can allocate an Elastic IP you have to reserve it


The IP has created and now ready to be allocated


Once it has been reserved you can associate with an instance


You can verify in the console


Now you could power down the instance and attach the Elastic IP to another instance.  I wont bother documenting but you get the point

 

Cloudwatch Metrics for EC2
Out of the box the following metrics for EC2 are gathered into Cloudwatch every 5 mins:

Disk IO - for Instance store backed AMIs

Network

CPU - for T instance types you also get the CPU credits

Host health - hypervisor hosting the EC2 instance

Guest health - EC2 instance Health

If you want to push RAM metrics into Cloudwatch you need to create a custom metric.  Custom. metrics can be more granular than 5 mins

To push custom metrics you have 2 options:

Install a Cloudwatch agent - this is what you would use to push logs into Cloudwatch (by default no logs are sent to Cloudwatch out of the box)

Use perl scripts to push the metrics

In order for the EC2 instances to be able to send logs and metrics to Cloudwatch they need to have the correct permissions assigned to them via Policies attached to the Role allocated to the instance

