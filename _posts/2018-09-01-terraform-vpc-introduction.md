---
published: false
---
## Learning AWS VPC essentials using Terraform

I recently passed the AWS Certified Solutions Architect Associate (CSAA) exam, mostly through following along with the video course from the guys at [A Cloud Guru](http://acloud.guru/). It's a great course and I recommend it to anyone wanting to take the certification. It's important for the exam to have a thorough understanding of VPC concepts and this is an area where I found just following the videos or working in the AWS console didn't really make the concepts click with me, as a coder I needed to get more hands-on.

Step forward [Terraform](https://www.terraform.io) an infrastructure-as-code tool similar to AWS Cloudformation but work with several cloud providers. The ability to define your infrastructure as code and gradually iterate on it and build it up piece-by-piece can give a better appreciation of the different parts involved.

Over the next few posts I'll walk through setting up a VPC and gradually building it up from something simple to a more complex system. To follow along you'll need your own AWS account and an access key / secret combination with admin permissions (or at least enough privileges to create and destroy the resources we'll be working with). Obviously you'll need to install Terraform as well and have it available in your path, installing the AWS CLI would be useful but not essential. For Windows users it will be useful to have Bash installed, more on that when it becomes useful.
