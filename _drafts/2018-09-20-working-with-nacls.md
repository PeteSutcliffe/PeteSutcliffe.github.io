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

Now we understand that theory, let's have a play. Create a new file nacl.tf and add the following:

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


