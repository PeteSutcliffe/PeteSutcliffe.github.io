---
published: false
title: Network Access Control Lists (NACLs)
---
Let's move on to NACLs. These are superficially similar to security groups but there are some key differences you need to be aware of for the exam.
Like security groups, NACLs define ingress and egress rules for traffic in your VPC. Where security groups are applied at the instance level, NACLs are applied at the network level and control all traffic into and out of a subnet. Every subnet must be associated with one (and only one) NACL either implicitly or explicitly. Every VPC must have a default NACL and one will automatically be created for you when you create a new VPC and any subnets that do not have a NACL explicitly set will use this default NACL.
