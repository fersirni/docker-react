Required AWS Elastic Beanstalk Environment Setup Updates - DO NOT SKIP
The process for creating an Elastic Beanstalk environment has seen major changes as of 4-1-2023. At the moment there are many undocumented and very buggy configuration requirements.

If you have a new AWS account, or, you've never used EC2 or EBS services in the past, then, you will need to set up a few things manually. If your AWS account is older, there is a chance that things like the default EBS and EC2 IAM roles have already been created.



The first thing you "might" need to do is to create an EC2 IAM role.

1. Go to AWS Management Console

2. Search for IAM and click the IAM Service.

3. Click Roles under Access Management in the left sidebar.

4. Click the Create role button.

5. Select AWS Service under Trusted entity type. Then select EC2 under common use cases.

6. Search for AWSElasticBeanstalk and select both the AWSElasticBeanstalkWorkerTier and AWSElasticBeanstalkMulticontainerDocker policies. Click the Next button.

7. Give the role the name of aws-elasticbeanstalk-ec2-role

8. Click the Create role button.



After you've created this EC2 role, you will need to create an EBS environment.

1. Go to AWS Management Console

2. Search for EBS and click the Elastic Beanstalk service.

3. If you've never used EBS before you will see a splash page. Click the Create Application button. If you have created EBS environments and applications before, you will be taken directly to the EBS dashboard. In this case, click the Create environment button. There is now a flow of 6 steps that you will be taken through.

5. You will need to provide an Application name, which will auto-populate an Environment Name.

6. Scroll down to find the Platform section. You will need to select the Platform of Docker. This will auto-populate the rest of the fields:


7.  Scroll down to the Presets section and make sure that free tier eligible has been selected:


8. Click the Next button to move to Step #2.

9. You will be taken to a Service Access configuration form.

If you are presented with a blank form where the Existing Service Roles field is empty, then, you should select Create and use new service role. You will need to set the EC2 instance profile to the aws-elasticbeanstalk-ec2-role created earlier (this may be auto-populated for you).

If both Existing Service Roles and EC2 Instance Profiles are populated with default values, then, select Use an existing service role.


10. Click the Skip to Review button as Steps 3-6 are not applicable.

11. Click the Submit button and wait for your new EBS application and environment to be created and launch.



After you have created an EBS environment, you may need to make a modification to the S3 bucket.

1. Go to AWS Management Console

2. Search for S3 and click the S3 service.

3. Find and click the elasticbeanstalk bucket that was automatically created with your environment.

4. Click Permissions menu tab

5. Find Object Ownership and click Edit

6. Change from ACLs disabled to ACLs enabled. Change Bucket owner Preferred to Object Writer. Check the box acknowledging the warning:


7. Click Save changes.

Required Updates for Docker Compose
This new AWS Linux 2 platform will conflict with the project we have built since it will look for a docker.compose.yml file to build from by default instead of a Dockerfile.

To resolve this, please do the following:

1. Rename the development Compose config file

Rename the docker-compose.yml file to docker-compose-dev.yml. Going forward you will need to pass a flag to specify which compose file you want to build and run from:
docker-compose -f docker-compose-dev.yml up
docker-compose -f docker-compose-dev.yml up --build
docker-compose -f docker-compose-dev.yml down

2. Create a production Compose config file

Create a docker-compose.yml file in the root of the project and paste the following:

version: '3'
services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '80:80'
AWS EBS will see a file named docker-compose.yml and use it to build the single container application.

https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/learn/lecture/27975358#overview