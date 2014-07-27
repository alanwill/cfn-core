These [AWS Cloudformation](http://aws.amazon.com/cloudformation/) templates were created to ease the creation of new accounts and VPCs and to ensure identical standards are followed. The intention, which is why it's called "core" is to provision the base network and access control infrastructure that the application builds upon.

Currently the templates will each provision a fully functional VPC that provides internet connectivity to private instances and has a network topology that spans two availability zones. They should run in all 8 AWS regions with the exception of GovCloud and Beijin and that can be easily remedied with a small ami-id addition. Specifically they do the following:

* Create a VPC with the following 5 subnets duplicated in 2-3 AZs (depending on the template) for logical network isolation of each application:
	* Internet Facing Load Balancer
	* Presentation
	* Internal Facing Load Balancer
	* Application
	* Database
* Provision a NAT instance in one of the Internet facing subnets and updates all yum packages on boot
* Attache an Internet Gateway
* Attache a Virtual Private Gateway for private connectivity via VPN or Direct Connect and enables route table propogation
* Create and configure a route table for internal subnets and another for internet subnets
* Tag all resources according to the values passed in via parameters
* Create a bastion instance to limit SSH access when connecting to application instances

You have complete control of the CIDRs that govern each subnet so the templates can be used to provision a network of virtually any size.

*** The only difference between core-2az.json and core-3az.json is that one provisions across 2 AZs and the other across 3 AZs otherwise they are identical. ***


##How to run
The template can be run either from the AWS Console or the CLI.

###GUI
This is probably the easiest way to run the script. Just download the [core.json](https://github.com/alanwill/cfn-core/blob/master/core.json) file from Github and upload it during the stack creation wizard. [Here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) is a walkthrough from the Cloudformation documentation.

###CLI
From the command line using the [AWS CLI](http://aws.amazon.com/cli/) run a command similar to the example below, with your parameters:

```
aws cloudformation create-stack \
--stack-name core \
--template-url https://s3.amazonaws.com/mybucket/core.json \
--parameters \
ParameterKey=ApplicationSubnetCidrAZ1,ParameterValue=10.43.10.128/26,UsePreviousValue=false \
ParameterKey=ApplicationSubnetCidrAZ2,ParameterValue=10.43.10.192/26,UsePreviousValue=false \
ParameterKey=AppName,ParameterValue=mycoolapp,UsePreviousValue=false \
ParameterKey=CorporateCidrIp,ParameterValue=10.0.0.0/8,UsePreviousValue=false \
ParameterKey=DatabaseSubnetCidrAZ1,ParameterValue=10.43.11.192/27,UsePreviousValue=false \
ParameterKey=DatabaseSubnetCidrAZ2,ParameterValue=10.43.11.224/27,UsePreviousValue=false \
ParameterKey=EnvironmentName,ParameterValue=dev,UsePreviousValue=false \
ParameterKey=InternalLoadBalancerSubnetCidrAZ1,ParameterValue=10.43.11.128/27,UsePreviousValue=false \
ParameterKey=InternalLoadBalancerSubnetCidrAZ2,ParameterValue=10.43.11.160/27,UsePreviousValue=false \
ParameterKey=InternetLoadBalancerSubnetCidrAZ1,ParameterValue=10.43.10.0/26,UsePreviousValue=false \
ParameterKey=InternetLoadBalancerSubnetCidrAZ2,ParameterValue=10.43.10.64/26,UsePreviousValue=false \
ParameterKey=NATKeyName,ParameterValue=infra-nat,UsePreviousValue=false \
ParameterKey=NATInstanceType,ParameterValue=t2.micro,UsePreviousValue=false \
ParameterKey=PresentationSubnetCidrAZ1,ParameterValue=10.43.11.0/26,UsePreviousValue=false \
ParameterKey=PresentationSubnetCidrAZ2,ParameterValue=10.43.11.64/26,UsePreviousValue=false \
ParameterKey=VPCCidr,ParameterValue=10.43.10.0/23,UsePreviousValue=false \
--capabilities CAPABILITY_IAM \
--on-failure ROLLBACK \
--region us-east-1
```


##Future Features
Once Cloudformation provides support for this, I'd like to be able to add:

* IAM Identity Provider for SAML
* IAM Roles with pre-specified names to work with SAML
* IAM account alias creation
* CloudTrail configured in all regions from a single template
* Import a public EC2 keypair

##Help
If something doesn't work as advertised or you have any questions or feedback, please submit a [Github issue](https://github.com/alanwill/cfn-core/issues/new).
