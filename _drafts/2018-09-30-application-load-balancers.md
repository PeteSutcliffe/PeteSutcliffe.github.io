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