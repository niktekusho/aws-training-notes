# AWS Elastic Load Balancing

Auto scaling ensures the customer has the right amount of EC2 instances to service the demand.

ELB = Load Balancing = Distributes incoming connections to 1 or more EC2 instances.

Scaling = Add resources on demand

Elasticity = Add or remove resources on demand

Scaling Up = Vertical Scaling = Add more resources to an instance (CPUs, Memory, Storage, Network bandwidth).

- Single Instance -> Single Point of Failure
- Scale by changing instance families/types (T2.micro to C5.xlarge)

Scaling Out = Horizontal Scaling = Add more instances

## EC2 Auto Scaling

Horizontal scaling of EC2 instances.

- EC2 Auto Scaling launches and terminates instances dynamically (scalability and elasticity)
- Auto Scaling can be triggered by EC2 Status Checks changes and CloudWatch metrics
- Can scale based on demand of the app (performance) or on a schedule
- Scaling Policy = Defines how to respond to changes in demand
- Autoscaling group = Defines collections of EC2 instances managed together (can be across Availability Zones)


### Auto Scaling Group

Used to create an auto scaling group:

1. Launch Template defines the EC2 instances configuration
   - AMI and instance type
   - EBS volumes
   - Security groups
   - Key pairs
   - IAM instance profile
   - User data
   - Shutdown behaviour
   - Termination protection
   - Placement Group name
   - Capacity reservation
   - Tenancy
   - Purchase Option (Spot, On-demand, etc)
2. Launch Config, like Launch Template but fewer options
   - AMI and instance type
   - EBS volumes
   - Security groups
   - Key pairs
   - IAM instance profile
   - User data
   - Purchase Option (Spot, On-demand, etc)

Group creation process:

1. Select Launch Template or Config
2. Configure purchase options
3. Configure VPC and subnets (INSIDE A REGION)
4. Attach Load Balancer
5. Configure health checks for EC2 instances and ELB
6. Define group size and scaling policies


- By default health checks are configured to look at the system status.
- ELB health checks use the ELB health check and EC2 instances status checks.
- Health check grace period = how long to wait before checking the health status of an instance


Auto Scaling monitoring:

1. ASG Metrics 
  - info sent to CloudWatch about the single group with 1 minute resolution (free)
  - must be enabled manually when creating the group or after
2. "Basic monitoring" (EC2 instances monitoring):
   - instances individually send info to CloudWatch every 5 minutes (free)
3. "Detailed monitoring" (EC2 instances):
   - instances individually send info to CloudWatch every 1 minute (paid)

Scaling policies settings:

- Cooldown = prevents Auto Scaling from launching and terminating instances before effects of previous operations are visible. Default = 300 seconds (5 mins)
- Termination Policy = Defines which instances are terminated first when a "shrink" occurs (Scale In)
- Termination Protection = Defines which instances cannot be terminated ("protected instances")
- Instance state = Defines the state of the instances. By default Auto Scaling puts instances in the `InService` state. Instances in `Standby` state are not terminated and can be inspected.
- Lifecycle hooks = Defines custom actions when Auto Scaling launches or terminates instances

Lifecycle hooks use cases:

1. run a script to download and install software. Custom lifecycle hooks puts instances into a paused state until confermation is received (via script itself or AWS Lambda)
2. pause an instance to process data before the termination


## ELB

Load Balancer should have a public/elastic IP to be used from outside.
When an instance fails, the load balancer reroutes the connection to a healthy instance -> High Availability

Fault tolerance = ability to recover from single component failure -> redundant components allow the system to continue operations

Availability Zones help in fault tolerance because they can be thought as separate data centers.

Auto Scaling + ELB allows to

1. always have the right number of instances
2. can recover from failures
3. elastically respond to service demand

### Types of ELBs


Application Load Balancer

- works at request level of HTTP and HTTPs (layer 7)
- supports path-based routing (**ATTENTION**: path is after /!), host-based routing (Host header), querystring-based routing and source IPs routing
- targets EC2 instances, IP addresses, Lambda functions and containers
- used with web applications, microservices architectures and Lambda targets

Target group = used to route requests to registered targets

Rule is configured on the listener (HTTP/HTTPS)

Network Load Balancer

- works at connection level of TCP, UDP, TLS, TCP_UDP, etc (layer 4)
- supports routing based on raw IP protocol data (?)
- targets UDP and static IP addresses
- used when ultra high performance, low late or TLS offloading (backend does not process TLS) is needed
- used when raw TCP and UDP are used
- used when you need to target a static IP address
- used when you need to interact with a VPC service endpoint

Listeners are required to listen to a unique port

Classic Load Balancer (Old gen, not recommended for new apps)

- supports routing based on data coming from layer 4 (connection protocol) or layer 7 (HTTP requests)
- used when supporting existing EC2 classic instances

Gateway Load Balancer

- supports routing based on data coming from layer 3 (connection packet)
- listens to all ports!
- forwards traffic to the "target groups" specified in the listener rules
- exchanges traffic with GENEVE protocol on port 6081
- used in front of firewalls, Intrusion Detection Systems (IDS)/Intrusion Protection Systems (IPS) and deep packets inspection systems


