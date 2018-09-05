---
published: false
---
## Creating our first public web server

We've created a VPC, let's put something in it to play with. In this post we'll create an EC2 instance, log onto it with SSH, install Apache and serve a simple web page from it.

To accomplish this we'll need to add the following to our VPC:
- An Internet Gateway (IGW)
- A subnet with a route to the IGW
- An EC2 instance
- A security group that allows incoming web and shh traffic
- A key pair to use to log on to our server

This is what we're going to build.

![VPC]({{site.baseurl}}//images/first_single_server.png)

For the key pair, if you already have one in AWS then you can reuse that, otherwise in the Console go to EC2 -> Key Pairs, create a new one and download it. Remember that key pairs are region specific so you'll need to create it in the same region you want to create your infrastructure. Windows users, check out this post on [using ssh in Window](http://pdsutcliffe.co.uk/2018-09-04/ssh-on-windows) if you are unfamiliar with using ssh.