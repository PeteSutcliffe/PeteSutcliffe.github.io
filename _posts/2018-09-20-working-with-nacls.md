---
published: false
title: Network Access Control Lists (NACLs)
---
Let's move on to NACLs. These are superficially similar to security groups but there are some key differences you need to be aware of for the exam.
Like security groups, NACLs define ingress and egress rules for traffic in your VPC. Where security groups are applied at the instance level, NACLs are applied at the network level and control all traffic into and out of a subnet. Every subnet must be associated with one (and only one) NACL either implicitly or explicitly. Every VPC must have a default NACL and one will automatically be created for you when you create a new VPC; any subnets that do not have a NACL explicitly set will use this default NACL. You can modify the default NACL or create a new one and make that the default, in which case you can delete the old one; however you can't delete one that is currently set as the default.

A NACL is made up of one or more numbered rules that define the traffic parameters and whether that traffic is allowed or denied. This differs from a security group that only lets you define allow rules. If you want to block a specific ip address for example, you could do this with a NACL but not a security group (exam tip...)
Rules are evaluated in ascending numerical order and once a matching rule is found no further rules are evaluated. If no rules match then there is a catch-all rule that denies access. So let's say you have the following rules defined:

|rule no|type|port|CIDR|EFFECT|
|-------|----|----|----|------|
|200|tcp|80|0.0.0.0/0|ALLOW|
|100|tcp|80|1.2.3.4/32|DENY|

If you were accessing from the ip address 1.2.3.4 using http port 80 then you would be denied access. Any other ip address would be allowed access via http, but not any other type of traffic. If you swapped the rule numbers around, then all http traffic would be allowed including from 1.2.3.4 because the ALLOW rule would match and evaluation would stop there.
It's good practice to increase your rules numbers in incremements of 100s so you've plenty of scope to insert rules without having to renumber your existing ones.

A note about the default NACL that is created with the VPC: it has a rule added for both ingress and egress that allows all traffic. This is the reason that we were able to access our instances even though we hadn't created any rules. If you create a new NACL though this rule is not added so the default behaviour of a custom NACL is to deny everything.

Now we understand that theory, let's have a play. Create a new file [nacl.tf](https://raw.githubusercontent.com/PeteSutcliffe/aws-vpc-terraform/787433b6036550cfafc8fb9a9b46892fa620f105/nacl.tf) and add the following:

``` HCL
resource "aws_network_acl" "main" {
  vpc_id = "${aws_vpc.vpc.id}"

  egress {
    protocol   = "all"
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  ingress {
    protocol   = "all"
    rule_no    = 100
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 0
    to_port    = 0
  }

  subnet_ids = ["${aws_subnet.public_subnet_a.id}"]

  tags {
    Name = "demo"
  }
}
```
We set up ingress and egress rules that allows all traffic to start with.
Apply the new infrastructure and confirm you can still access the index page using the server ip address.
Let's make things more restrictive by just allowing http traffic. Change the ingress rule to protocol "tcp" from_port and to_port to 80 then apply again and verify the index page is still accessible.
Now do the same thing for the egress and apply again. Try the index page again and... the page is inaccessible, what happened? If you were to enable VPC flow logs and look at the denied traffic you'd see requests coming from your ip to the server but they don't come from port 80, they come from an [ephemeral port](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports). When your PC makes a request to a web site, it chooses a port for the request to come back on. This makes sense if you think about it, if your PC is a web server itself and a request comes back in on port 80, it would try to handle it as an inbound web request. 
In order to allow our subnet to reply to incoming requests we need to open the full range of possible ports requesting clients might use: 1024-65535. Let's change our egress rule to from_port 1024 and to_port 65535. Access should now be restored.

That's serving inbound requests dealt with, how about making outbound ones? Try outbound.php, it's probably no suprise that this isn't working currently. There are 2 things we need to do to fix it, based on our experience with the incoming requests can you work out what they are?
The first is obvious, we aren't allowing outbound traffic on port 80 so our http request isn't allowed out of the subnet, we can fix this with a new rule:

``` HCL
egress {
    protocol   = "tcp"
    rule_no    = 200
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 80
    to_port    = 80
  }
```
This allows traffic to reach the external server but isn't sufficient by itself, we also need to let the reply back in, which will be coming back on, you guessed it, an ephemeral port. According to the documentation linked above, Amazon Linux uses ports 32768-61000 so let's add a new ingress rule:
``` HCL
ingress {
    protocol   = "tcp"
    rule_no    = 200
    action     = "allow"
    cidr_block = "0.0.0.0/0"
    from_port  = 32768
    to_port    = 61000
  }
```
This should see outbound.php working again.

So that's network access control lists, what a pain. You might be wondering why we didn't have to bother with ephemeral ports when setting up security groups; this comes back to security groups being stateful and NACLs being stateless. If an inbound or outbound request is allowed by the relevant ingress or egress rules in a security group, the return traffic is allowed as well, regardless of ports. This isn't the case for NACLs where ingress and egress rules have to be explicitly set. This is important to remember for the exam; you might be given a scenario and asked to choose the correct answer from security groups or NACLs with similar looking rules, you need to take into account the different behaviours of the 2 to work out the correct answer.

Next time we'll look at adding a bit of sophistication to our setup with a load balancer.
