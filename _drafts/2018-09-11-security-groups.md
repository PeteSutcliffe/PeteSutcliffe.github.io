---
published: false
title: Working with Security Groups
---
Security groups are basically firewall rules that control what traffic can come into and out of your instances. A security group with no rules will block all traffic, rules are then added in to allow specific types of traffic on specific ports from a particular source. e.g. to allow http traffic (tcp) on port 80 from any ip address. Interestingly security groups can serve as sources for other security groups so you could apply a security group to a load balancer then set another security group on your ec2 instances to only allow http traffic from the load balancer's security group. Unlike NACLs, security groups only have ALLOW rules so can't be used to block traffic from a specific source such as an attacker's ip address.

With that out of the way let's play around with our rules and see what happens. Recall that our current security group looks like this:

``` HCL

resource "aws_security_group" "demo_sg" {
  name        = "demo_sg"
  description = "Allow inbound http / ssh traffic"

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  vpc_id = "${aws_vpc.vpc.id}"
}

```

We have 3 rules, one lets http traffic in on port 80, one lets ssh traffic in on port 22 and one let's all traffic out of the instance.
Make sure your instance is up and running and that you can access the index.html in a web browser using the ip address of the instance, and the outbound.php file successfully.

Now, delete or comment out (use a hash at the start of each line) the ingress rule for port 80. Now run `terraform apply` and Terraform will update the rule without changing any other infrastructure. You should now find that both pages will time out if you try to access them. However, you can still ssh onto the server and once on the server you can still access the web e.g. using curl or ping.
Now let's reverse the situation. Restore the deleted rule and replace the egress rule with the following: `egress = []`. We can't just remove out the rule because if there are no rules defined Terraform will think we don' want to change the egress rules and leave them as they are.
Apply this change and our index page will now be accessible but the outbound page times out. Our server can answer http requests but is unable to make any outbound calls. Similarly your ssh session will no longer be able to access the web.

There's an important takeaway from this that might not be immediately obvious; even when we don't explicitly allow outbound traffic the security group will allow http traffic out when it originates from an inbound request. Similarly, when egress is allowed, return traffic from an outgoing http call is allowed back in even if an inbound rule is not configured. This is because security groups are _stateful_ and is a key thing to understand for the exam. This will become clearer when we look at Network Access Control Lists (NACLs), which are _stateless_ next time.