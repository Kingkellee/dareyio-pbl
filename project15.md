# AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY

## Project Objective

In this project I mannually built a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company called `Kolture `. `Kolture` uses WordPress CMS for its main business website, and a [Tooling Website](https://github.com/Kingkellee/tooling-1) for their DevOps team.

As part of Kolture desire for improved security and performance, a decision has been made to use a reverse proxy technology from `NGINX` to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accomodate to increased traffic and, at the same time, has reasonable cost.

## Project Architecture at a Glance

#

![](images/project15/tooling_project_15.png)

## Starting Off Your AWS Cloud Project

#

- Properly configure your AWS account and Organization Unit, create a Root account in AWS
- Within the root account, create a sub-account and name it DevOps. (You will need another email address to complete this)
  ![](images/project15/organizations-root.png)
  ![](images/project15/organization-DevOps.png)
- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there)
- Move the DevOps account into the Dev OU.
  ![](images/project15/organization-dev.png)
- Login to the newly created AWS account using the new email address.
- Create a free domain name for your fictitious company at Freenom domain registrar here.
- Create a hosted zone in AWS, and map it to your free domain from Freenom.
  ![](images/project15/hosted-zone.png)

## Set Up a Virtual Private Network (VPC)

#

## Step 1. Create a VPC

#

- Search for VPC, and click on `Create a VPC`
  ![](images/project15/create-vpc.png)
- Fill accordingly, Input the required `IPv4 CIDR Block`
  ![](images/project15/vpc-created.png)

## Step 2. Create subnets as shown in the architecture

#

- Create `Public Subnets 1` as shown below
  ![](images/project15/create-public-subnet.png)
- Create `Public Subnets 2` as shown below
  ![](images/project15/create-public-subnet2.png)
- Create `Private Subnets 1` as shown below
  ![](images/project15/create-private-subnet.png)
- Create `Private Subnets 2` as shown below
  ![](images/project15/create-private-subnet1.png)
- Create `Private Subnets 3`
- Create `Private Subnets 4` as shown below
  ![](images/project15/create-private-subnet4.png)

Subnets
![](images/project15/subnets.png)

## Step 3. Create a route table and associate it with public subnets

#

- Create `Public Route Table`
- Edit Subnet Association and Assign `Public Subnets 1` and `Public Subnets 2` to the Public Route Table
  ![](images/project15/route-table-subnetasociations.png)

- Create `Private Route Table` and associate it with the private subnets 1, 2, 3 and 4

## Step 4. Create Internet Gateway

#

- From the `VPC Dashboard`, click on `Internet Gateway`, Click on `Create Internet Gateway` and Attach the `Internet Gateway` to the `VPC`
  Internet Gatewway
  ![](images/project15/create-igw.png)
- Edit a route in the public rouere table and associate it with the `Internet Gateway`
  ![](images/project15/routes-for-public-subnet.png)

## Step 5. Create a `Nat Gateway`.

#

Nat Gateway
![](images/project15/NAT-created.png)

- Allocate an Elastic IP to the NAT Gateway
  ![](images/project15/allocate-EIP-ngw.png)

## Step 6. Create `Security Groups`

#

- `External Application Load Balancer` Security Group

  - Allow Http and Https access from Internet
    ![](images/project15/sg-external-load-balancer.png)

- `Nginx Reverse Proxy Server` Security Group

  - Allow SSH from Bastion, HTTP and HTTPS from `External Application Load Balancer`
    ![](images/project15/sg-nginx.png)

- `Bastion Servers` Security Group

  - Allow SSH from only the workstation pulic IP address, use `curl www.canhazip.com` to get your workstation IP
    ![](images/project15/sg-baston.png)

- `Internal Application Load Balancer` Security Group

  - Allow Http and Https Access through the `NGINX reverse proxy`
    ![](images/project15/sg-internal-load-balancer.png)

- `Webservers` Security Group

  - Allow SSH Access from `Bastion`, Http and Https through the `Intenal Application Load Balancer`
    ![](images/project15/sg-webservers.png)

- `Data Layer` Security Group
  - Allow MYSQL/Aurora access through the `Webservers`
  - Allow MYSQL/Aurora acdcess through the `Bastion`
  - Allow NFS access through the `Webservers`

Security Goups
![](images/project15/sg-all-created.png)

## Step 7. Transfer Domain to `Route 53` Hosted Zone and Create AWS Certificate Manager

#

- **Creating an Hosted Zone**

  - Choose Get started under DNS management or choose Hosted zones in the navigation pane.
  - Choose Create hosted zone.
  - In the Create Hosted Zone pane, enter the name of the domain that you want to route traffic for. You can also optionally enter a comment.
  - For Type, accept the default value of Public Hosted Zone.
  - Choose Create.
  - Create records that specify how you want to route traffic for the domain and subdomains.
    ![](images/project15/hosted-zone.png)

  - **Create AWS Certificate Manager**
    - Serach for `AWS Certificate Manager` in AWS Services
    - Request a public wildcard certificate for the domain name you registered in Freenom
      ![](images/project15/acm-request-public-certificate.png)
    - Fil acordingly
      ![](images/project15/acm-dns-validation.png)
      ![](images/project15/acm-select-key-alog.png)
    - Goto Validation, Use DNS to validate the domain name
      ![](images/project15/acm-domain-validation.png)
    - Click on Create Route in `Route 53` to verify
      ![](images/project15/acm-issued-certificated.png)

## Step 8. AWS Elastic File System

#

- In AWS Services, Search for `Elastic File System`, Click on `Create File System`
  ![](images/project15/efs-create-file-system.png)

- Fill Accordingly

  - Select the VPC
    ![](images/project15/efs-step1.png)
  - Select the Mount Target in the Availability Zones
    - Mount the efs on Private Subents 1 and 2 to the Data Layer Security Group
      ![](images/project15/efs-step2.png)
  - Click Next and Leave Defaults for other Option
    ![](images/project15/efs-step3.png)
  - Click on Create

- Configure the Access Points
  - Cick on `Access Points`
    ![](images/project15/efs-step5.png)
  - Click on File System, Create Accesspoint for the File system as show below
    ![](images/project15/efs-step6.png)
    ![](images/project15/efs-step7.png)
    ![](images/project15/efs-step8.png)
  - Repeat the Process for `tooling` with route path as `/tooling`

## Step 9. AWS Key Management Service

#

- Navigate to AWS ACM
  - Click on `Create a Key`
  - Request a public wildcard certificate for the domain name you registered in Freenom
    ![](images/project15/kms-step1.png)
    ![](images/project15/kms-step2.png)
  - Use DNS to validate the domain name
  - Tag the resource
    ![](images/project15/kms-created-key.png)

## Step 10. Create a `Relational Database Service`

#

- Goto Amazon RDS
  - Select Subnet Groups
  - Create a DB Subnet Group for `RDS` by selecting the Datalayer Security-Group in Private Subnets 3 and 4
    ![click on create](images/project15/rds-create-subnet-group.png)
  - Fill Accordingly
    ![](images/project15/rds-add-subnets.png)
    ![](images/project15/rds-created-subnet.png)
- Go Back to Dahboard
- Click Datatabases
- Click Create Databases
- Select MySQL
  ![](images/project15/rds-create.png)
- Select Free Tier in Key Management System
- Edit Database Configuration Settings as shown
  ![](images/project15/rds-create-db.png)
- Select the right `VPC` and `RDS SUBNET GROUP`
- Select `Datalayer Security Group`
  ![](images/project15/rds-select-sg.png)
- Create Test databse
  ![](images/project15/rds-db-additional-config.png)
- Click Create
  ![](images/project15/rds-database-created.png)

## Set Up Compute Resources

#

## Step 11. Compute Resource for Bastion

#

- Create an EC2 Instance (RHEL) for Bastion in the default VPC
- Launch into the EC2 instances
- Associate an Elastic IP withthe Bastion EC2 Instances
- Change to root user
  ```
  sudo su
  ```
- Configure the Instance with the following Software
  ```
    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y
    systemctl start chronyd
    systemctl enable chronyd
  ```

## Step 12. Compute the Resource for NGINX

#

- Create an EC2 Instance (RHEL) for NGINX in the Default VPC
- Launch into the EC2 instances
- Change to root user
  ```
  sudo su
  ```
- Configure the Instance with the following Software
  ```
    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y
    systemctl start chronyd
    systemctl enable chronyd
  ```
- Configure selinux policies for the webservers and nginx servers
  ```
    setsebool -P httpd_can_network_connect=1
    setsebool -P httpd_can_network_connect_db=1
    setsebool -P httpd_execmem=1
    setsebool -P httpd_use_nfs 1
  ```
- Install amazon efs utils for mounting the target on the Elastic file system
  ```
    git clone https://github.com/aws/efs-utils
    cd efs-utils
    yum install -y make
    yum install -y rpm-build
    make rpm
    yum install -y  ./build/amazon-efs-utils*rpm
  ```
- Set up self-signed certificate for the nginx instance
  ```
    sudo mkdir /etc/ssl/private
    sudo chmod 700 /etc/ssl/private
  ```
  ```
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/Kolture.key -out /etc/ssl/certs/Kolture.crt
  ```
  ```
    sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
  ```
  ![](images/project15/openssl-certificate.png)

## Step 13. Compute the Resource for Webserver

#

- Create an EC2 Instance (RHEL) to be used by Wordpress and Tooling Launch Template in the Default VPC
- Launch into the EC2 instances
- Change to root user
  ```
  sudo su
  ```
- Configure the Instance with the following Software
  ```
    yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
    yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm
    yum install wget vim python3 telnet htop git mysql net-tools chrony -y
    systemctl start chronyd
    systemctl enable chronyd
  ```
- Configure selinux policies for the webservers
  ```
    setsebool -P httpd_can_network_connect=1
    setsebool -P httpd_can_network_connect_db=1
    setsebool -P httpd_execmem=1
    setsebool -P httpd_use_nfs 1
  ```
- Install amazon efs utils for mounting the target on the Elastic file system
  ```
    git clone https://github.com/aws/efs-utils
    cd efs-utils
    yum install -y make
    yum install -y rpm-build
    make rpm
    yum install -y  ./build/amazon-efs-utils*rpm
  ```
- Set up self-signed certificate for the apache webserver instance
  ```
  yum install -y mod_ssl
  openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/Kolture.key -x509 -days 365 -out /etc/pki/tls/certs/Kolture.crt
  vi /etc/httpd/conf.d/ssl.conf
  ```
  > Edit ...localhost.key and ..localhost.crt to Kolture.key and Kolture.crt

## Step 14. Configure Target Groups

#

- **Baston Server Target Group**

  - Select Instances as the target type
  - Ensure the protocol is TCP on port 22
  - Register Bastion Instances as targets
  - Ensure that health check passes for the target group

- **NGINX Target Group**

  - Select Instances as the target type
  - Ensure the protocol HTTPS on secure TLS port 443
  - Ensure that the health check path is `/healthstatus`
  - Register Nginx Instances as targets
  - Ensure that health check passes for the target group

- **Webservers Target Group**
  - Select Instances as the target type
  - Ensure the protocol HTTPS on secure TLS port 443
  - Ensure that the health check path is `/healthstatus`
  - Register Webserver Instances as targets
  - Ensure that health check passes for the target group

Target Groups
![](images/project15/targetgroup-all.png)

## Step 15. Create AMI

#

- On the Launched Instance, click on `Actions`, Select `AMI and Images`
- Create an AMI out of the Bastion EC2-Instance
  ![](images/project15/ami-create-jumpbox.png)
- Create an AMI out of the NGINX EC2-Instance
  ![](images/project15/ami-create-nginx.png)
- Create an AMI out of the Webserver EC2-Instance
  ![](images/project15/ami-create-webserver.png)

AMI-For Launch Templates  
![](images/project15/ami-created-all.png)

## Step 16. Configure Launch Templates

#

- **Prepare Launch Template For Bastion (One Per Subnets)**

  - Navigate to Launch Templates and click on `Create Launch Template`
    ![](images/project15/LT-step1.png)
  - Provide Appropriate Template name and Description
    ![](images/project15/LT-step2.png)
  - Make use of the Baston AMI to set up the launch template
    ![](images/project15/LT-step3.png)
  - Attach Key Pairs
    ![](images/project15/LT-step4.png)
    Ensure the Instances are launched into a public subnet
  - Assign appropriate security group
    ![](images/project15/LT-step6.png)
    Configure Userdata to update yum package repository and install Ansible and git
    ![](images/project15/LT-step7.png)

- **Prepare Launch Template For NGINX (One Per Subnets)**

  - Navigate to Launch Templates and click on `Create Launch Template`
    ![](images/project15/LT-step1.png)
  - Provide Appropriate Template name and Description
    ![](images/project15/LT-nginx-step-2.png)
  - Make use of the NGINX AMI to set up the launch template
    ![](images/project15/LT-nginx-step1.png)
  - Attach Key Pairs
    ![](images/project15/LT-step4.png)
    Ensure the Instances are launched into a public subnet
  - Assign appropriate security group
    ![](images/project15/LT-nginx-step2.png)
    Configure Userdata to update yum package repository and install nginx
    ![](images/project15/nginx-userdata.png)

- **Prepare Launch Template For Wordpress Server (One Per Subnets)**

  - Navigate to Launch Templates and click on `Create Launch Template`
    ![](images/project15/LT-step1.png)
  - Provide Appropriate Template name and Description
    ![](images/project15/lt-wordpress.png)
  - Make use of the Baston AMI to set up the launch template
  - Attach Key Pairs
    ![](images/project15/LT-step4.png)
    Ensure the Instances are launched into a public subnet
  - Assign appropriate security group
    ![](images/project15/LT-wordpress-step2.png)
  - From EFS, Click on the File System, Click on Access Point and copy the access point in wordservers, edit the Userdata with the right mount point
  - Configure Userdata to update yum package repository and install Ansible and git
    ![](images/project15/wordpress-userdata.png)

- **Prepare Launch Template For Tooling (One Per Subnets)**

  - Navigate to Launch Templates and click on `Create Launch Template`
    ![](images/project15/LT-step1.png)
  - Provide Appropriate Template name and Description
    ![](images/project15/LT-tooling-step1.png)
  - Make use of the NGINX AMI to set up the launch template
  - Attach Key Pairs
    ![](images/project15/LT-step4.png)
    Ensure the Instances are launched into a public subnet
  - Assign Webserver Security group
  - From EFS, Click on the File System, Click on Access Point and copy the access point in tooling, edit the Userdata with the right mount point
    ![](images/project15/efs-tooling-mount.png)
  - Configure Userdata to update yum package repository and install nginx
    ![](images/project15/tooling-userdata.png)

![](images/project15/LT-all-created.png)

Launch Templates

## Step 17. Configure Application Load Balancer (ALB)

#

- **Application Load Balancer To Route Traffic To NGINX**

  - Create an Internet facing ALB
    ![](images/project15/alb-create.png)
    ![](images/project15/alb-ext-step1.png)
  - Ensure the ALB is created within the appropriate VPC | AZ | Subnets
    ![](images/project15/alb-ext-step2.png)
  - Ensure that it listens on HTTPS protocol (TCP port 443)
  - Select the `External Load Balancer` Security Group
    ![](images/project15/alb-ext-step3.png)
  - Choose the Certificate from ACM
  - Select Nginx Instances as the target group

- **Application Load Balancer To Route Traffic To Web Servers**
  - Create an Internal ALB
    ![](images/project15/alb-create.png)
    ![](images/project15/alb-int-step1.png)
  - Ensure the ALB is created within the appropriate VPC | AZ | Subnets
    ![](images/project15/alb-int-step2.png)
  - Select `Internal Load Balancer` Security Group
  - Ensure that it listens on HTTPS protocol (TCP port 443)
    ![](images/project15/alb-int-step3.png)
  - Choose the Certificate from ACM
    ![](images/project15/alb-int-step4.png)
  - Select Wordpress Server Instances as the target group
  - Click Create
  - Select internal load balancer and click on listeners, Click on Manage Rules
    ![](images/project15/alb-int-step6.png)
  - Configure a rule to forward tooling request to the tooling target
    ![](images/project15/internal-load-balancing-rules.png) and save Rules

Application Load Balancers
![](images/project15/alb-all-created.png)

## Step 18. Configuring Autoscalling Group

#

- **Autoscaling Group for Bastion**
- Select the `Bastion launch template`
- Select the VPC
- Select both `Public subnets`
  ![](images/project15/AG-step1.png)
- Bastion Server need no Application Load Balancer for the AutoScalingGroup (ASG)
- Ensure that you have health checks for both EC2 and ALB
  ![](images/project15/AG-step2.png)
- The desired capacity is 1
- Minimum capacity is 1
- Maximum capacity is 1
- Set scale out if CPU utilization reaches 90%
  ![](images/project15/AG-step3.png)
- Ensure there is an SNS topic to send scaling notifications
  ![](images/project15/AG-step4.png)
- Review and Create Autoscaling Group

- **Autoscaling Group for NGiNX**
- Select the `NGINX launch template`
- Select the VPC
- Select both public subnets
  ![](images/project15/AG-step1.png)
- Enable Application Load Balancer for the AutoScalingGroup (ASG)
- Select the NGINX target group
- Ensure that health checks for both EC2 and ALB
  ![](images/project15/AG-step2.png)
- The desired capacity is 1
- Minimum capacity is 1
- Maximum capacity is 1
- Set scale out if CPU utilization reaches 90%
  ![](images/project15/AG-step3.png)
- Ensure there is an SNS topic to send scaling notifications
  ![](images/project15/AG-step4.png)
- Review and Create Autoscaling Group

- **Autoscaling Group for WordPress Server**
- Select the `Webserver launch template`
- Select the VPC
- Select both `Private subnets 1 and 2`
  ![](images/project15/AG-wordpress.png)
- Enable Application Load Balancer for the AutoScalingGroup (ASG)
- Select the `Wordpress target group`
- Ensure that you have health checks for both EC2 and ALB
  ![](images/project15/AG-step2.png)
- The desired capacity is 1
- Minimum capacity is 1
- Maximum capacity is 1
- Set scale out if CPU utilization reaches 90%
  ![](images/project15/AG-step3.png)
- Ensure there is an SNS topic to send scaling notifications
  ![](images/project15/AG-step4.png)

- **Autoscaling Group for Tooling Server**
- Select the `Webserver launch template`
- Select the VPC
- Select both Private Subnets 1 and 2
  ![](images/project15/AG-tooling.png)
- Enable Application Load Balancer for the AutoScalingGroup (ASG)
- Select the `Tooling target group`
  ![](images/project15/AG-tooling-LB.png)
- Ensure that you have health checks for both EC2 and ALB
  ![](images/project15/AG-step2.png)
- The desired capacity is 1
- Minimum capacity is 1
- Maximum capacity is 1
- Set scale out if CPU utilization reaches 90%
  ![](images/project15/AG-step3.png)
- Ensure there is an SNS topic to send scaling notifications
  ![](images/project15/AG-step4.png)
- Review and Create Autoscaling Group
- Review and Create Autoscaling Group

## Step. 19. Health Checks

#

- **Ensure that health check passes for the NGINX target group**
  ![](images/project15/TG-healthy-nginx.png)
- **Ensure that health check passes for the WordPress target group**
  ![](images/project15/healthy-wordpress-target-group.png)
- **Ensure that health check passes for the Tooling target group**
  ![](images/project15/healthy-tooling-target-group.png)

## Step 20: Result

#

Entering the url `https://www.wordpress.kingkellee.ga` or `wordpress.kingkellee.ga` will use nginx reverse proxy to route traffic from the External ALB to the wordpress site through the internal ALB:
![](images/project15/wordpress.png)

Entering the url `https://www.tooling.kingkellee.ga` or `tooling.kingkellee.ga` will use nginx reverse proxy to route traffic from the External ALB to the tooling site through the internal ALB:
![](images/project15/tooling.png)
