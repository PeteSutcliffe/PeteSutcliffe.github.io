---
published: false
---
## Creating our first public web server

We've created a VPC, let's put something in it to play with. In this post we'll create an EC2 instance, log onto it with SSH, install Apache.
To accomplish this we'll need to add the following to our VPC:
- An Internet Gateway (IGW)
- A subnet with a route to the IGW
- An EC2 instance
- A security group that allows incoming web and shh traffic
- A key pair to use to log on to our server