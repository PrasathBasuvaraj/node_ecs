### Requirements to  start a node project:
    
    ---install node js
    ---install npm
### To start a node project:
 --- create a directory and open the directory in Vs code:
 --- Run the command "npm init" to start a node project
 ---and run the command "npm install" to install dependencies
 ---add a index.js file which is the file gets started when running a project
 ---Run the command "node inde.js" which will run the project in the respective envirnments.
   http://localhost:3000

### To build a image in Docker:

      "docker build -t node-image ."

## to run that image in a container:
      "docker container run -d -p 3000:3000 node-image"

### Steps To push the image to ECR:

   --- To login Of ECR through DOCKER is 
       "aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 169064206213.dkr.ecr.ap-south-1.amazonaws.com"
   
   --- To create a repo in AWS ECR:
      "aws ecr create-repository --repository-name node-repo --region ap-south-1"

   --- To give a tag to your image:
      "docker tag node-image:latest 169064206213.dkr.ecr.ap-south-1.amazonaws.com/node-repo:latest"


   --- To Push the image to ECR:
      "docker push 169064206213.dkr.ecr.ap-south-1.amazonaws.com/node-repo:latest"

  --- To pull the image from ECR:
      "docker pull 169064206213.dkr.ecr.ap-south-1.amazonaws.com/node-repo:latest"

  --- To run the Image in the ECR Locally:
      "docker run -d -p 3000:3000 169064206213.dkr.ecr.ap-south-1.amazonaws.com/node-repo:latest"


### To create VPC and Related Resources:

   --- First Create A VPC "  aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications  'ResourceType=vpc,Tags=[{Key=Name,Value=NODEVPC}]'  "
  
   ---Next  Create SUBNETS FOR THE VPC:
       aws ec2 create-subnet --vpc-id vpc-0f672d8c6e7fbfd41 --cidr-block 10.0.1.0/24 --availability-zone ap-south-1a
       aws ec2 create-subnet --vpc-id vpc-0f672d8c6e7fbfd41 --cidr-block 10.0.2.0/24 --availability-zone ap-south-1b

   --- Next create  INTERNET GATEWAY:
       "aws ec2 create-internet-gateway --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=NODEINTERNETGATEWAY}]'   "

   --- Next  attach the internet gateway to VPC:
        "aws ec2 attach-internet-gateway --internet-gateway-id igw-04a0353ec94fd87b1 --vpc-id vpc-0f672d8c6e7fbfd41"
   
   --- Next create ROUTE TABLE:
       "aws ec2 create-route-table --vpc-id vpc-0f672d8c6e7fbfd41 --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=NODEROUTETABLE}]'"

   ---Create a ROUTE to the INTERNET GATEWAY:
      "aws ec2 create-route --route-table-id rtb-0c5f0b69ee1649e27 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-04a0353ec94fd87b1"
   
   ---Next associate route tables with Subnets:
      "
       aws ec2 associate-route-table --route-table-id rtb-0c5f0b69ee1649e27 --subnet-id  subnet-05d63f1558dfe8565
       aws ec2 associate-route-table --route-table-id rtb-0c5f0b69ee1649e27 --subnet-id subnet-0a57431efe1cca044
       "
   
   --- To create a security Group:
      "aws ec2 create-security-group --group-name ecs-sg --description "Security group for ECS with NODE" --vpc-id vpc-0f672d8c6e7fbfd41"


### To create a ECS CLUSTER:
   ---aws ecs create-cluster --cluster-name node-ecs-cluster
### Register the Task_definition.json to the ECS
   ---  aws ecs register-task-definition --cli-input-json file://task-definition.json

### To create SERVICE in ECS CLUSTER:
  ---  aws ecs create-service --cluster node-ecs-cluster --service-name node-service --task-definition node-app --desired-count 2 --launch-type FARGATE --network-configuration "awsvpcConfiguration={subnets=[subnet-05d63f1558dfe8565,subnet-0a57431efe1cca044],securityGroups=[sg-08317181751c7169f],assignPublicIp=ENABLED}"


  ### Run the application with the endpoint:

     http://13.127.15.133:3000/