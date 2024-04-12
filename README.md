# Serverless Disaster Recovery with AWS Global Accelerator Demo

This repository contains a demo showcasing features of AWS Services. When launching the CloudFormation template you will be able to see and test, how to adopt a serverless disaster recovery method with AWS Global Accelerator, Amazon Elastic Load Balancers, AWS Lambda and Amazon DynamoDB Global tables.

## Requirements

* **AWS Account:** If you don’t have an account nor an account provided to you, [click here](https://aws.amazon.com/es/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc).
* **Text editor (optional):** Just to make things easier and more organised.

## Architecture

![Demo Architecture](/images/serverless-dr-ga)

The architecture deployed is comprised of 2 VPCs: 1 in eu-west-1 and the other in us-east-1. This simulates an active-active multi-region deployment with a serverless architecture.

## Launch the template

2. Deploy the AWS Cloud Formation template by downloading the "vpc-first-region.yaml" file located in the "Templates" folder in this repo.

NOTE: This template will be deployed in Ireland region and it will take 5 min. to deploy approx.

3. Once the first template has been deployed, go ahead and deploy the second, by downloading the "vpc-second-region.yaml" file located in the "Templates" folder in this repo. Make sure to be on N.Virginia before deploying it.

In the second template, in the parameters section, make sure to enter the Application Load Balancer ARN. You will find it in the outputs section of the CloudFormation template deployed in Ireland:

![Template1 outputs](/images/template1-output.png)

NOTE: This template should be deployed in N.Virginia region and it will take 10 min. to deploy approx.

## Explore your environment

Each VPC has 4 subnets (2 publics and 2 privates). In the public ones there is a NAT Gateway to give private resources the capability to reach the internet (the best practice is to have a NAT Gateway per AZ). 

Remember, with AWS Global Accelerator you are provided 2 global static public IPs that act as a fixed entry point to your application, improving availability. These entry points (anycast IPs) are assigned to Elastic Network Interfaces (ENI) that are deployed, in our case, in the private subnets (one in each AZ). These ENIs will route the traffic to the Application Load Balancer (ALB) which is going to forward it to the backend, hosted on AWS Lambda.

AWS Lambda function retrieves the data from an Amazon DynamoDB Global table that automatically replicates the changes among the tables spread around the globe. In this case the tables are in Ireland and N.Virginia regions. 

## Test the architecture deployed

When the 2 AWS CloudFormation templates are deployed, go to the second template to the outputs section and copy and paste on your browser the DNS of the AWS Global Accelerator recently created. Depending upon your location, your request is going to be forwarded to the closest region (between ireland and N.Virginia):

![Template2 outputs](/images/template2-output.png)

And the result you'll see is this one:

![Backend-ireland](/images/backend-ireland.png)

As you can see, it displays the data from the infrastructure deployed in Ireland because both regions are healthy so the traffic is going to be routed to the closest location. 

Now, let's do a test, and delete an ALB of the Ireland region to see what happens:

 ![Backend-virginia](/images/backend-virginia.png)

As there is no ALB in Ireland, there is no way that packet can reach the backend, so the infrastructure in that region is not healthy, that's why the traffic will go through AWS backbone to N.Virginia.
 

## Clean up your environment

After completing the demo, delete AWS CloudFormation Stack using AWS Console (first delete the second template) or AWS CLI. If you're using the CLI, enter these commands in order:

```bash
aws cloudformation delete-stack --stack-name $STACKNAME2
```
Once the second template is deleted, input the same command with the first template

```bash
aws cloudformation delete-stack --stack-name $STACKNAME1
```

NOTE: The second template will take time to delete as the AWS Global accelerator distribution takes time to delete.


Cheers!!

## Authors

* Daniel Neri
* Serhat Gülbetekin

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.

