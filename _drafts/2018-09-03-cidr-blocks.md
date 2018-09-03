---
published: false
---
## Understanding CIDR block ranges

When creating a VPC we are required to supply a CIDR block range as a parameter. In the sample I chose 10.0.0.0/16 but hat does this mean and what value should we choose? Well, it turns out there are lots of values we _could_ choose but only a handful that we _should_.

As you probably know, an IPv4 address is a 32 bit number, which for convenience we break up into 4 blocks of 8 bits and write in the form of a.b.c.d where each letter is a number between 0 and 255. So in the example above 10.0.0.0 is recognizable as an ip address, what does the /16 mean? This represents a range of ip addresses. 