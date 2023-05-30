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
