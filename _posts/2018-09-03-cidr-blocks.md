---
published: true
title: Understanding CIDR block ranges
---
When creating a VPC we are required to supply a CIDR block range as a parameter. In the sample I chose 10.0.0.0/16 but hat does this mean and what value should we choose? Well, it turns out there are lots of values we _could_ choose but only a handful that we _should_ use.

As you probably know, an IPv4 address is a 32 bit number, which for convenience we break up into 4 blocks of 8 bits and write in the form of a.b.c.d where each letter is a number between 0 and 255. So in the example above 10.0.0.0 is recognizable as an ip address, what does the /16 mean? This notation is used to represent a range of ip addresses, the number representing how many bits (starting from the left) are _fixed_. So the smaller the number the bigger the range is.

It's easiest to think about it in blocks of 8, let's look at some simple examples.
10.0.1.0/24 means the first 24 bits (or 3 blocks) are fixed, with the last 8 changeable so it represents the ip range 10.0.1.0 - 10.0.1.255. Similarly 10.1.0.0/16 means the first 2 blocks are fixed so we get the range 10.1.0.0 - 10.1.255.255. We aren't limited to neat multiples of 8, for example a /31 address represents a range of just 2 addresses. A /32 address represents a single fixed address. 0.0.0.0/0 represents all possible addresses and is used a lot in security groups and NACLs in AWS to represent "the internet", or traffic from anywhere in other words.

So what should you pick? The value we choose defines the private ip addresses our resources will be assigned inside the VPC, while technically you could choose any valid CIDR block range (up to /16 which is the maximum AWS allows) there are [3 blocks set aside for use as private ips](https://en.wikipedia.org/wiki/Private_network) which you should choose from:
```
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```
As I said, /16 is the largest range AWS will let you have so you can't use the full range of the first 2 for your VPC.

Why should you choose one of these? To avoid potential conflicts. As long as you choose one of these, you can be sure that the private ips on your network won't clash with any public ip addresses on the internet and your traffic will be reliably routed to the correct destination.

**Exam tip:** VPC peering allows 2 VPCs from the same or different accounts to be connected and talk to each other over their private networks but this is only possible as long as the 2 VPCs don't have any overlap in their CIDR block ranges.
