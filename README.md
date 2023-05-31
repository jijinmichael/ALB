## Application Load Balancer

In AWS, ALB stands for Application Load Balancer. It is a service that provides high availability and automatic scaling for your applications. ALB operates at the application layer (Layer 7) of the OSI model and can distribute incoming traffic to multiple targets, such as EC2 instances, containers, or Lambda functions, within a region.

ALB is a powerful service that helps improve the availability, scalability, and fault tolerance of your applications. It plays a crucial role in distributing traffic efficiently across multiple targets and simplifies the management of your infrastructure.

In this post we are going to see canary deployment vs blue green deployment of the applications.

Canary deployment and blue-green deployment are both deployment strategies used in software development and release processes. They are designed to minimize risks and downtime during application updates or releases.

### Canary Deployment
---
- Canary deployment involves rolling out a new version of an application to a subset of users or servers, while the majority of the traffic still goes to the stable version.

- The new version, also known as the "canary," is initially exposed to a small percentage of users or servers to gather feedback and monitor its performance.

- By gradually increasing the exposure, the development team can closely observe the canary's behavior and assess its impact on the system.

- If the canary version proves to be stable and performs well, more traffic can be directed to it until it eventually replaces the old version.

- Canary deployments allow for quick rollbacks if issues are detected since only a small portion of users or servers are affected.

- This strategy provides an incremental and controlled approach to validating new features or changes before full-scale deployment.

### Blue Green Deployment
---
- Blue-green deployment involves running two identical environments, one called "blue" (the current stable version) and the other called "green" (the new version).

- Initially, all traffic is directed to the blue environment while the green environment remains idle.

- The new version is thoroughly tested and validated in the green environment, which is identical to the blue environment.

- Once the green environment is deemed stable, traffic is switched from the blue environment to the green environment in a single switchover.

- Blue-green deployments allow for a fast and seamless transition from the old version to the new version without any downtime or disruption for users.

- If any issues are encountered in the green environment, traffic can be quickly routed back to the blue environment, enabling easy rollbacks.


In summary, canary deployment involves gradually exposing a new version to a subset of users or servers, while blue-green deployment involves running two identical environments and switching traffic between them. Canary deployment provides a controlled and incremental approach, while blue-green deployment enables seamless switchover with minimal downtime. The choice between the two strategies depends on factors such as the level of risk tolerance, the size of the user base, and the specific requirements of the application.

To demonstrate this, I need 3 versions of the applications. Here I'm writing a simple small index pages with 3 versions like version-1, version-2 and version-3. To make those please follow the below steps.

#### Step-1
---
- Create a launch configuration

![image](https://github.com/jijinmichael/ALB/assets/134680540/8da8487b-0dad-4545-89df-97fb228cd562)

Here I'm assigning the name for launch configuration as **shopping-app-version-1** with Amazon AMI, it's ID is **ami-0607784b46cbe5816** with instance type **t2.micro**.

![image](https://github.com/jijinmichael/ALB/assets/134680540/444b328a-1806-4157-b562-354e67e30d07)

Under the additional details give the user data as follows.
```
#!/bin/bash


echo "ClientAliveInterval 60" >> /etc/ssh/sshd_config
echo "LANG=en_US.utf-8" >> /etc/environment
echo "LC_ALL=en_US.utf-8" >> /etc/environment
systemctl restart sshd.service

yum install httpd php -y

cat <<EOF > /var/www/html/index.php
<?php
\$output = shell_exec('echo $HOSTNAME');
echo "<h1><center><pre>\$output</pre></center></h1>";
echo "<h1><center>shopping-app-version-1</center></h1>"
?>
EOF


systemctl restart php-fpm.service httpd.service
systemctl enable php-fpm.service httpd.service
```
Add the required security group, key pair then create the LC.

#### Step-2
---
- Create Auto Scaling Group for the LC

![image](https://github.com/jijinmichael/ALB/assets/134680540/be96006e-6d79-44f6-abc1-12cd12d54abf)

- **ASG name** : shopping-app-version-1
- **LC**       : shopping-app-version-1

![image](https://github.com/jijinmichael/ALB/assets/134680540/78a3a2f1-175f-4d33-8e6f-1fadf6bdf702)

Select the VPC and Availability Zones and subnets. Here I'm going with default VPC and AZ ap-south-1a and ap-south-1b.

On the Configure group size and scaling policies, mention Group size as follows.
- **Desired capacity** = 2
- **Minimum capacity** = 2
- **Maximum capacity** = 2

At the last step mention a tag name as version-1 to identify the instance created by this ASG.

![image](https://github.com/jijinmichael/ALB/assets/134680540/dc9e640a-5963-4ca1-8ff8-13e000c6bd00)

#### Step-3
---
- Create a target group

![image](https://github.com/jijinmichael/ALB/assets/134680540/cfadc9b1-2ea0-4c3f-8997-b1e658b1bfc0)

- **Choose a target type** : instances
- **Target group name**    : shopping-app-version-1

Here I'm giving the **Health check path** : /index.php

Leave the rest of the field as it is.

#### Step-4
---
- Assign ASG to TG(target Group)

Go to the ASG >> Select the ASG >> Edit. And assign the TG to ASG as below.

![image](https://github.com/jijinmichael/ALB/assets/134680540/6d4853d0-c722-4d7d-bc47-78e68707440a)

#### Step-5
---
- Create ALB

Now we are creating the application load balancer. 

![image](https://github.com/jijinmichael/ALB/assets/134680540/1ed55b58-0cf5-47f3-929a-e45bda769a33)

From here click on the ALB section.

Create ALB with following details.
- **Load balancer name** : shopping-app
    - On the Network mapping select the VPC and AZ's. 
    - Assign the required SG.
    - Listener : Change it to HTTPS and add the shopping-app-version-1 TG
    - Load the cert from ACM and create the ALB as follows.

![image](https://github.com/jijinmichael/ALB/assets/134680540/c9bea178-ebf0-4eca-a655-d30ac685bc11)

![image](https://github.com/jijinmichael/ALB/assets/134680540/ffa714c0-7408-464a-8f14-519f8d37b92e)

![image](https://github.com/jijinmichael/ALB/assets/134680540/074f5de7-5998-495c-8a53-e20fd932eecc)

![image](https://github.com/jijinmichael/ALB/assets/134680540/b6d8a7dd-85cd-4424-ab48-2cc44b944b93)

#### Step-6
---
Then we need to edit the listener to redirect the HTTP request to HTTPS. For this, select the above created TG >> Listener tab >> Add listener.

![image](https://github.com/jijinmichael/ALB/assets/134680540/fc821253-b97f-4adf-ae0f-b07bd294a066)

And make the chnages as follows.

![image](https://github.com/jijinmichael/ALB/assets/134680540/f7d0e008-073e-4944-a257-ac278cbd0d0b)

Again we need to make some changes in the listener. So select the HTTPS:443 protocol in the listener and click on the actions >> Manage Rules.

You will get an interface like below.

![image](https://github.com/jijinmichael/ALB/assets/134680540/fbf921da-621b-440a-8d5a-029809fb41d6)

On this click on the edit button on the right top and delete the default rule and add the entry like below.

![image](https://github.com/jijinmichael/ALB/assets/134680540/1d303aa5-7df3-4160-8fab-6d65b0846b24)

Then add a host header for the domain like as per yours and forward it to the TG which we created previously.

![image](https://github.com/jijinmichael/ALB/assets/134680540/610dfa71-33aa-4780-97ec-2a6df4755323)

#### Step-7
---
- Map the record to the ALB in Route53

Click on Route 53 >> DNS management >> Click on Hosted Zone >> Select the Hosted Zone >> Create Record as follows.

![image](https://github.com/jijinmichael/ALB/assets/134680540/428ae0c7-e961-4bfd-b84e-a24559a83b60)

![image](https://github.com/jijinmichael/ALB/assets/134680540/2c496ebd-fe02-4b5c-9b98-b1a7d8e632f7)

![image](https://github.com/jijinmichael/ALB/assets/134680540/c04c2b86-efdd-4a95-8ac4-a45be574586a)

Then you will get the site as below from the 2 instances since we deployed 2 instances in our ASG.

![image](https://github.com/jijinmichael/ALB/assets/134680540/c8f56316-e56f-4538-8030-5bfd0fb6cb3d)

![image](https://github.com/jijinmichael/ALB/assets/134680540/46947a37-a4e6-4cdd-a416-a496398a8e21)

Now I want to change the version-1 of the application to version-2 with out any down time with 90% traffic to version-1 and 10% traffic to the version-2. In this case we are doing canary deployment. Please note in order to do so the LB must support canary deployment. ALB is such a one which is capable of it.

<p align="center">
  <img src="https://github.com/jijinmichael/ALB/assets/134680540/aca921a8-0a02-4980-967f-2e44564ad63d"></p>









