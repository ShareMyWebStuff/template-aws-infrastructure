# cloudformation-aws-react-mysql-template

## Introduction to Project

This is the first repository in a set of repositories to create a serverless react application running on AWS. The react application will allow clients to register and login. Once a client has successfully entered their details to register they will receive an email. This email will contain a link, this link when clicked will notify our application that the clients email address they entered is valid. Once the clients email address is validated the client will be able to securely logon and logout.

This project will detail how to run scripts that set the AWS infrastructure up, explain where manual intervention is required (for instance pointing an external domain name to this web site), deploy sql code and data loads to the mySQL database, setup the backend lambda functions and finally walk throught the react application. Each of these will occur in a different repo and each repo will explain what is contained in it.

This template is created based on a personal project Im currently working on sharemytutoring.com.

This project consists of the following repositories.
1.  template-aws-infrastructure
2.  template-mysql-setup
3.  template-lambda-layer
4.  template-lambda-functions
5.  template-react-application

This repositories should be viewed in the order they are listed above.


### Repositories

#### template-aws-infrastructure

This repository will contain all the cloudformation scripts to create your infrastructure. You may have heard of the term Infrastructure As Code (IaC), the scripts in this repository create your infrastructure. The beauty of these scripts as they can be run for every new project and always create the exact same setup.

#### template-mysql-setup

This repository contains all the database scripts to create the tables, stored procedures and data uploads. This repository will explain how to deploy these to your database.

#### template-lambda-layer

This repository contains the code to create a lambda layer. This is basically common code the lambda functions use such as database assess. These layers reduce the size of the lambda functions and hence reduce the cold start time.

#### template-lambda-functions

This repository contains the code to create the lambda functions as well as the api gateway. This is basically common code the lambda functions use, such as database assess. These layers reduce the size of the lambda functions and hence reduce the cold start time of the lambda functions .

#### template-react-application

This repository contains the react code for this project.


### Assumptions

This project makes the following assumptions
1.  You have an understanding of AWS and the components this project will use.
2.  You have an AWS account
3.  You have created a key pair (explained later)
4.  You have purchased a domain name (mine is hosted by 1 and 1)

**Finally, to create this project on AWS you need to understand that it will predominently be covered by free tier but some of the setup may mean you occur charges.**


## Create the Infrastructure (this repository)

The AWS infrastructure is created via the following repository. This repository consists of several cloudformation stacks which can either be manually deployed via the AWS cloudformation console or via the command line. The command line allows CI/CD to be setup to automatically deploy our infrastructure.

The repository contains several stacks. The stacks can be used to create two different setups

1.  S3 bucket containing react application with a mySQL database. Domain name points to S3 buckets.
2.  S3 bucket containing react application with a mySQL database. The S3 buckets has a cloudfront distribution that is accessed via a domain name.

Both setups have the same VPC and database. The difference lies in how our domain name accesses the website. The first one has a domain name that points to S3 buckets (via the hosting zone). The second has a domain name that points to cloudfront, the cloudfront distribution has lambda edge functions to add security headers as well as converting the communication from HTTP to HTTPS (for security), the cloudfront then points to an S3 bucket, has a SSL certificate associated to it, the S3 bucket does not have public access as the first setup does. The second setup is more complex and may cost a little more, however, there is a major advantage because of the extra security added. If this is a production setup then use the second option.

The stacks in this repo are
create-vpc
create-mysql-rds
create-hosting-zone
create-bucket-hosting
create-certificate
create-cloudfront-lambda-edge-fns
create-cloudfront-hosting
create-public-ec2

Each of these stacks is described later on.

### Prerequisites

Before creating your infrastructure you will need an AWS account and to have created a key pair. The key pair creation is described below. There are also some stacks which will need manual interaction either whilst they are running or after they have run, these are create-hosting-zone and create-certificate. The create-hosted-zone will need to have the domain name pointed to the AWS nameservers on the hosted zone after it has run, this is described below. The create-certificate will not complete until you add a record it specifies in the hosted zone, this is required so AWS knows you own the domain.

#### Create EC2 Key Pair

In the AWS console, goto EC2. Down the left hand side under network and security select "Key Pairs". When you create a key pair the pem 
file will be automatically downloaded to your download directory. To create a pem file to connect to the EC2 instance.
1.  Select "Create Key Pair"
  1. Ordered sub-list
  1.    Enter the name. For instance "sharemytutoring-kp"
  1.    Select the file format (pem).
  1.    Click "Create Key Pair". 
2.  The sharemytutoring-kp.pem file will automartically be downloaded. Keep this in a safe place as it will be used to connect to any EC2 
instances you create.

1. First ordered list item
2. Another item
  * Unordered sub-list. 
1. Actual numbers don't matter, just that it's a number
  1. Ordered sub-list
4. And another item.


#### Once Hosting Zone is Created

Once the hosting zone is created we need to point our domain names DNS name servers to AWS. 
From the AWS console navigate to Route 53. On the left hand navigation bar select hosted zones and select the domain name and domain extension 
you selected ( mine was sharemytutoring.info ). If you click on this hosting zone you will see the various DNS entries. You will see a NS ( name 
server) record, this will have four entries. Each of these entries needs to be updated on the name servers on where your doamin is registered.
I use 1and1 hence
1.  Logon onto 1and1
2.  Select Domains & SSL
3.  Select Nameserver
4.  Edit Nameserver and enter a row for each of the AWS name servers.

https://dnschecker.org/ can be used to see if the name servers have changed.

## Once Create Certificate is Running

Once the create certificate is running a message will appear as per below

Content of DNS Record is: {Name: _3452345243fasdf6212fda9fba85433.sharemytutoring.co.uk.,Type: CNAME,Value: _wefsfsf3223423sddfwer53f.rgsdgsdfgsdffg.acm-validations.aws.}

Navigate to the AWS Route 53 created and in the hosted zone create a record set as per below. Do not click alias.
Name: _3452345243fasdf6212fda9fba85433
Type: CNAME
Value: _wefsfsf3223423sddfwer53f.rgsdgsdfgsdffg.acm-validations.aws

### Create S3 Bucket Hosting

The S3 bucket hosting setup requires the following stacks.
create-vpc
create-mysql-rds
create-hosting-zone
create-bucket-hosting

### Create Cloudfront Hosting

To create a cloudfront setup the following stacks are required. 
**Two of these stacks need to be run out of the US East ( N. Virginia) region as this is a requirement for certificates and lambda edge functions. The others should be run in the same region you want to run them in.**

create-vpc
create-mysql-rds
create-hosting-zone
create-certificate (us-east-1)
create-cloudfront-lambda-edge-fns (us-east-1)
create-cloudfront-hosting

## Stack Descriptions

This section describes each stack. 


## create-vpc.yml

This cloudformation file creates all the items for a VPC. The VPC will have public and private subnets. 


### Parameters

| Parameter Name      | Default Value    | Parameter Description                                                                                             |
| --------------      | -------------    | ---------------------                                                                                             |
| DomainName:         | sharemytutoring. | This contains just the domain name without the domain extension.                                                  |
| DomainExtension:    | com.             | This contains the domain extension for instance co.uk, co, com etc.                                               |


### AWS Items Created

Once this cloudformation file is uploaded it will create the following infrastructure. This creates the networking and security aspect of our project. 

The following is VPC related

| Item              | Name                                 | Description                                                                            |
| ----              | ----                                 | ------------                                                                           |
| VPC               | sharemytutoring-vpc                  | The VPC for this project                                                               |
| Internet Gateway  | sharemytutoring-igw                  | THE IGW is attached to the VPC to give it internet connectivity.                       |
| Subnet            | sharemytutoring-public-subnet-A      | The public subnet in the VPC.                                                          |
| Subnet            | sharemytutoring-private-subnet-A     | The first private subnet in the VPC.                                                   |
| Subnet            | sharemytutoring-private-subnet-B     | The second public subnet in the VPC.                                                   |
| Route Table       | sharemytutoring-public-route-A       | The public route table A associated with public subnet A.                              |
| Route Table       | sharemytutoring-private-route-A      | The private route table associated with private subnet A.                              |
| Route Table       | sharemytutoring-private-route-B      | The private route table associated with private subnet B.                              |
| ACL               | Default ACL name                     | The ACL applied to all subnets.                                                        |
| Security Group    | sharemytutoring-fe-sg                | The public subnet security group. Allows internet connectivity and ssh capability.     |
| Security Group    | sharemytutoring-be-sg                | The private subnet security group. Allows mysql connectivity to public security group. |
| Security Group    | sharemytutoring-lambda-sg            | The lambda security group.                                                             |

## create-mysql-rds

This cloudformation file creates the mysql database and database subnet. The envuironment parameter will create a repliocated database if set to production.


### Parameters

| Parameter Name      | Default Value           | Parameter Description                                                                                      |
| --------------      | -------------           | ---------------------                                                                                      |
| Environment:        | Dev                     | Whether we create a development or production spec database.                                               |
| DomainName:         | sharemytutoring.        | This contains just the domain name without the domain extension.                                           |
| DomainExtension:    | com.                    | This contains the domain extension for instance co.uk, co, com etc.                                        |
| StackNameVPC:       | sharemytutoring-com-vpc | This is the name of the create-vpc stack. This will fail without the create-vpc stack run in the same      |
|                     |                         | region.                                                                                                    |
| DBServerSize:       | db.t2.micro             | Free tier server.                                                                                          |
| DBServerName:       | sharemytutoring         | The name of your database server (it does not need to be the same name as the domain).                     |
| DBDatabaseName:     | sharemytutoring         | The name of your database, it can be called anything.                                                      |
| DBUsername:         | Admin                   | The username of the database user.                                                                         |
| DBPassword:         | Tester01                | The password for the DBUsername.                                                                           |

### AWS Items Created

Once this cloudformation file is uploaded it will create the mySQL database and database subnet.

| Item              | Name                                 | Description                                                                            |
| ----              | ----                                 | ------------                                                                           |
| DB Subnet Group   | sharemytutoring-db-subnet-group      | Specifies the subnets the database can access.                                         |
| DB Instance       | sharemytutoring-db                   | The MySQL DB instance.                                                                 |

## create-hosting-zone

This cloudformation file creates the DNS hosted zone for your domain name. Once this is run you will need to forward your domains nameservers to point to the AWS nameservers. This is in the prerequisite section.

### Parameters

| Parameter Name      | Default Value           | Parameter Description                                                                                      |
| --------------      | -------------           | ---------------------                                                                                      |
| DomainName:         | sharemytutoring.        | This contains just the domain name without the domain extension.                                           |
| DomainExtension:    | com.                    | This contains the domain extension for instance co.uk, co, com etc.                                        |

### AWS Items Created

Once this cloudformation file is uploaded it will create the route53 hosted zone.

| Item              | Name                                 | Description                                                                            |
| ----              | ----                                 | ------------                                                                           |
| Hosted Zone       | sharemytutoring.com                  | Creates the hosting zone for the domain.                                               |


## create-bucket-hosting

This cloudformation file creates the S3 buckets for your website and adds records to the Route53 recordset to point to these buckets. If this is run for sharemytutoring.com it will create two buckets, one called sharemytutoring.com and another www.sharemytutoring.com. The www.sharemytutoring.com bucket is redirected to sharemytutoring.com which is where you will need to deploy your react code, a record set will be added for each of these.

### Parameters

| Parameter Name      | Default Value           | Parameter Description                                                                                      |
| --------------      | -------------           | ---------------------                                                                                      |
| StackNameHostedZone |                         | This is the name of the hosted zone stack previously run.                                                  |
| DomainName:         | sharemytutoring.        | This contains just the domain name without the domain extension.                                           |
| DomainExtension:    | com.                    | This contains the domain extension for instance co.uk, co, com etc.                                        |

### AWS Items Created

Once this cloudformation file is uploaded it will create the route53 hosted zone.

| Item              | Name                                 | Description                                                                            |
| ----              | ----                                 | ------------                                                                           |
| Bucket            | sharemytutoring.info                 | Creates the S3 bucket for the react website.                                           |
| Bucket Policy     |                                      | Creates the read only bucket policy for sharemytutoring.info.                          |
| Bucket            | www.sharemytutoring.info             | Creates the S3 bucket for the react website and redirects it to the bucket above.      |
|                   |                                      | This means the react code is only deployed to one bucket.                              |
| Bucket Policy     |                                      | Creates the read only bucket policy for www.sharemytutoring.info.                      |
| Route53 Recordset |                                      | Adds a record to the DNS for sharemytutoring.info.                                     |
| Route53 Recordset |                                      | Adds a record to the DNS for www.sharemytutoring.info.                                 |


## create-certificate

This cloudformation file creates the SSL certificate for your website.
**This needs to be run in the N. Virginia region.**

### Parameters

| Parameter Name      | Default Value           | Parameter Description                                                                                      |
| --------------      | -------------           | ---------------------                                                                                      |
| DomainName:         | sharemytutoring.        | This contains just the domain name without the domain extension.                                           |
| DomainExtension:    | com.                    | This contains the domain extension for instance co.uk, co, com etc.                                        |
| HostedZoneID        |                         | This contains the hosted zone id we created earlier on.                                                    |

### AWS Items Created

Once this cloudformation file is uploaded it will create the route53 hosted zone.

| Item              | Name                                 | Description                                                                            |
| ----              | ----                                 | ------------                                                                           |
| Certificate       |                                      | Creates a certificate thats held in the N. Virginia region.                            |


## create-cloudfront-lambda-edge-fns

This cloudformation file creates the lambda edge functions that add header records returned to the clients browsers. 
** This needs to be run in the N. Virginia region.

The following headers are returned.
  strict-transport-security   ensures future connections are over https and not http
  x-content-type-options      MIME types in Content-Type are not changed. Stops MIME type sniffinh
  x-frame-options             Stops our site being embedded in <frame>, <iframe>, <embed> or <object>. Stops clickjacking attacks.
  content-security-policy     Helps guard against cross site scripting attacks.
  x-xss-protection            IE, Chrome and Safari will stop loading if they detect cross-site scripting
  referrer-policy             Referrer will be sent if the same origin

### Parameters

| Parameter Name      | Default Value           | Parameter Description                                                                                      |
| --------------      | -------------           | ---------------------                                                                                      |
| DomainName:         | sharemytutoring.        | This contains just the domain name without the domain extension.                                           |
| DomainExtension:    | com.                    | This contains the domain extension for instance co.uk, co, com etc.                                        |

### AWS Items Created

None

## create-cloudfront-hosting

This creates the cloudformation distribution thats linked to the certificate, links to the lambda edge functions, creates the S3 bucket thats contains your react code and adds the route 53 record to associate the domain to the cloudfront distribution.

### Parameters

| Parameter Name      | Default Value           | Parameter Description                                                                                      |
| --------------      | -------------           | ---------------------                                                                                      |
| StackNameHostedZone |                         | This is the name of the stack we ran earlier that created the hosted zone.                                 |
| DomainName:         | sharemytutoring.        | This contains just the domain name without the domain extension.                                           |
| DomainExtension:    | com.                    | This contains the domain extension for instance co.uk, co, com etc.                                        |
| LambdaEdgeArn:      | com.                    | The ARN of the lambda edge function created in the N. Virginia region.                                     |
| CertificateArn:     | com.                    | The ARN of the certificate created in the N. Virginia region.                                              |

### AWS Items Created

Once this cloudformation file is uploaded it will create the route53 hosted zone.

| Item              | Name                                 | Description                                                                            |
| ----              | ----                                 | ------------                                                                           |
| S3 Bucket         | sharemytutoring.com                  | Creates the bucket to hold the react code. Bucket can be named anything.               |
| S3 Bucket Policy  |                                      | Creates the bucket policy so we can only read it.                                      |
| cloudfront        |                                      | Creates a cloudfront distrubution. You may want to change this to suit your needs.     |
| Route 53 Recordset|                                      | Adds the recordset to connect the domain name and cloudfront distribution.             |

## create-public-ec2

This cloudformation file creates an EC2 instance, this instance allows us to connect to the database when we need to create items such as tables, stored procedures and data loads. The EC2 instance can be sshed through by mySQL workbench. The EC2 instance can be recreated or stopped and started when you want to deploy database items. 

**Putty**

AWS creates a PEM file for the key pair, putty needs a ppk (private key) file. The ppk can be generated from the pem file using puttygen.
1.  Download putty.
2.  Open PuttyGen
3.  Click the load button, set the file type to "*" and select the AWS PEM file.
4.  Click save private key.

After running this script and the successful creation of the EC2 instance. Putty can be configured to connect to this EC2 server.

1.  Find the AWS EC2 instance on AWS console. 
2.  Select the instance and click the connect button at the top of the screen next to the Launch Instance button. This will display the connection details.
3.  Get the user name and IP address of the EC2 instance from the connection screen.
4.  Open the putty connect screen.
  1.    Enter the IP address on the Session tab
  2.    On the connection->Data screen enter the EC2 user name. Usually ec2-user.
  3.    On the Connection-> SSH -> Auth screen browse to the ppk we creted earlier.
  4.    Save the connection details and click Open.
5.  You should now be connected to the EC2 Instance.

**MySQL Workbench SSH PassThrough Setup**

Once the EC2 Instance is set

1.  Start MySQL workbench on your local computer
2.  Select Database -> Manage Connections from the menu at the top.
3.  Goto your AWS MySQL database and get its end point e.g. sharemytutoring.<unique number>.eu-west-2.rds.amazonaws.com
4.  Set the follow inputs on the Manage Server Connection from Workbench
  1.    Connection Method   Standard TCP/IP over SSH
  2.    SSH Hostname        The IP address of your EC2 instance
  3.    SSH Username        The ec2 username e.g. ec2-user
  4.    SSH Key File        Browse and select the AWS PEM file we generated (KeyPair)
  5.    MySQL Hostname      This is the end point we found at 3.
  6.    MySQL Server Port   3306 - unless you have changed the port number.
  7.    Username            Admin - or the user name you created when creating the database
5.  Click test connection at the bottom of the screen

You should now be able to connect the the MySQL database via the EC2 instance


### Parameters

| Parameter Name      | Default Value           | Parameter Description                                                                                      |
| --------------      | -------------           | ---------------------                                                                                      |
| DomainName:         | sharemytutoring.        | This contains just the domain name without the domain extension.                                           |
| DomainExtension:    | com.                    | This contains the domain extension for instance co.uk, co, com etc.                                        |
| HostedZoneID        |                         | This contains the hosted zone id we created earlier on.                                                    |

### AWS Items Created

Once this cloudformation file is uploaded it will create the route53 hosted zone.

| Item              | Name                                 | Description                                                                            |
| ----              | ----                                 | ------------                                                                           |
| EC2 Instance      |                                      | Creas an EC2 instance.                                                                 |


