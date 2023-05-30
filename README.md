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

First of all let us do a canary deployment. In order to proceed with this, please follow the below steps.

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

- Create Auto Scaling Group for the LC

![image](https://github.com/jijinmichael/ALB/assets/134680540/be96006e-6d79-44f6-abc1-12cd12d54abf)

**ASG name** : shopping-app-version-1
**LC**       : shopping-app-version-1

![image](https://github.com/jijinmichael/ALB/assets/134680540/78a3a2f1-175f-4d33-8e6f-1fadf6bdf702)

Select the VPC and Availability Zones and subnets. Here I'm going with default VPC and AZ ap-south-1a and ap-south-1b.

On the Configure group size and scaling policies, mention Group size as follows.
 - **Desired capacity** = 2
 - **Minimum capacity** = 2
 - **Maximum capacity** = 2

At the last step mention a tag name as version-1 to identify the instance created by this ASG.

![image](https://github.com/jijinmichael/ALB/assets/134680540/dc9e640a-5963-4ca1-8ff8-13e000c6bd00)



