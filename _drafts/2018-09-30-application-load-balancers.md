---
published: false
title: Application load balancers
---
Let's start to make our infrastructure a bit more robust. We'll start simple by adding an extra server in a different availability zone and putting a load balancer in front of them. This will bring us 2 benefits: if a calamity befalls the data centre of one our servers is in, our application will still be reachable through the server in the other one, also our capacity to serve traffic will increase as we have an extra server.

First up we'll need another subnet to put our server in:

``` HCL
resource "aws_subnet" "public_subnet_b" {
  vpc_id                  = "${aws_vpc.vpc.id}"
  cidr_block              = "10.0.2.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "Subnet pub eu-west-1b"
  }
}
```
This one is in eu-west-1b and has a different CIDR block range from the one in zone 1a. With our net subnet in place we'll need to give that subnet a route to the IGW to enable traffic to / from the internet.

``` HCL
resource "aws_route_table_association" "public_subnet_b_association" {
  subnet_id      = "${aws_subnet.public_subnet_b.id}"
  route_table_id = "${aws_route_table.internet_route.id}"
}
```

Let's add an EC2 instance into the new subnet:
``` HCL
resource "aws_instance" "server2" {
  ami                    = "ami-0bdb1d6c15a40392c"
  instance_type          = "t2.micro"
  vpc_security_group_ids = ["${aws_security_group.demo_sg.id}"]
  key_name               = "pete-eu-west-1"
  subnet_id              = "${aws_subnet.public_subnet_b.id}"
  user_data              = "${file("userdata.txt")}"

  tags {
    Name = "Web Server2"
  }
}
```
Finally let's add the various bits we need to set-up the load balancer: loadbalancer.tf
``` HCL
resource "aws_lb" "demo" {
  name               = "demo-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = ["${aws_security_group.demo_sg.id}"]
  subnets            = ["${aws_subnet.public_subnet_a.id}", "${aws_subnet.public_subnet_b.id}"]

  enable_deletion_protection = false
}
```

We are going to use an Application Load Balancer, which will probably be the default choice in most cases unless there's a good reason to go with one of the others. It's set up to balance external traffic across the 2 subnets we are using. For convenience I've reused the same security group as the web servers.

Next up we define a target group for the loadbalancer and add our 2 instances to it:
``` HCL
resource "aws_lb_target_group" "demo" {
  name     = "demo-lb-tg"
  port     = 80
  protocol = "HTTP"
  vpc_id   = "${aws_vpc.vpc.id}"

  health_check {
    interval            = 5
    timeout             = 2
    path                = "/healthcheck.html"
    matcher             = "200"
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}

resource "aws_alb_target_group_attachment" "server1_attachment" {
  target_group_arn = "${aws_lb_target_group.demo.arn}"
  target_id        = "${aws_instance.server.id}"
  port             = 80
}

resource "aws_alb_target_group_attachment" "server2_attachment" {
  target_group_arn = "${aws_lb_target_group.demo.arn}"
  target_id        = "${aws_instance.server2.id}"
  port             = 80
}
```
We'll forward traffic to our instances on port 80 over http. A health check is configured that is set to succeed and fail quickly. It will ping our healthcheck.html endpoint every 5 seconds and mark the instance as healthy if it returns 200 twice and will mark it as unhealthy and remove it from service if it fails twice. Each server is added to the target group individually.
Lastly we need to set up a listener:
``` HCL
resource "aws_lb_listener" "alb_listener" {
  load_balancer_arn = "${aws_lb.demo.arn}"
  port              = "80"
  protocol          = "HTTP"

  default_action {
    target_group_arn = "${aws_lb_target_group.demo.arn}"
    type             = "forward"
  }
}
```
We'll listen to requests over http on port 80. We just need a default rule, which is to forward all requests to our target group. If we needed to setup more complex rules, such as path-based routing we could do so by adding `aws_lb_listener_rule` resources.
This is all we need to get everything up and running but to make life a bit easier we'll make 2 more changes. Firstly add ingress and egress rules to the NACL rules so we can SSH onto our servers, secondly let's add the 2nd server's ip and the loadbalancer dns to the outputs:

``` HCL
output "server2_ip" {
  value = "${aws_instance.server2.public_ip}"
}

output "alb_dns" {
  value = "${aws_lb.demo.dns_name}"
}
```

Apply the changes and all being well you should get 2 server outputs and a dns address in the output. Take each ip and paste them into the browser address bar and you should get 2 different instance ids in the output. Do the same with the dns address and you should get alternating instance ids as you refresh the page (depending on the browser this might not work as we didn't specify any cache behaviour for the page so some may choose to cache the result, a tool such as Postman should give the expected output).

So let's experiment a bit. SSH onto one of the servers and go to where our pages are stored: `cd /var/www/html` now rename the index file: `sudo mv index.html index.bak`. Now try refreshing the browser with the alb dns address. Notice that half the requests give an error page, this won't fix itself as only the healthcheck page will affect server availability. Put the index page back: `sudo mv index.bak index.html`. Now rename the healthcheck page: `sudo mv healthcheck.html healthcheck.bak`. After a few seconds the alb dns will stop returning results from this server, however the page can still be reached by using the server's ip address directly. Load balancer healthchecks will not terminate unhealthy instances, nor start new ones in their place, they simply stop serving traffic to them. Put the healthcheck file back `sudo mv healthcheck.bak healthcheck.html` and after a few seconds the server will start getting requests from the load balancer again.

We've made our infrastructure a bit more robust but there's still a lot of improvements we can make. Next time we'll look at serving our web traffic from private instances through a NAT gateway.

