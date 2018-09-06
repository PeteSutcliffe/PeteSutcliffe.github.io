---
published: true
title: Getting started with Terraform
---
Let's make a start and create our VPC. First let's look at the basic of working with Terraform. I'd encourage you to work through the [getting started](https://www.terraform.io/intro/getting-started/install.html) guide on the Terraform site for more detailed guidance.

Firstly create a folder to put your Terraform files in. You can use any text editor to work with Terraform files but choosing one with specialist support will make life easier. I use [Visual Studio Code](https://code.visualstudio.com/), which has a Terraform plugin that will do autocomplete, reformatting, linting and other useful tricks.

Terraform will automatically process any files with a .tf extension in the directory it's run from allowing us to organize the resources we will create easily. 

Let's create a new file: [**vpc.tf**](https://raw.githubusercontent.com/PeteSutcliffe/aws-vpc-terraform/b7358083734b2c1e236d62025ac980ca7c0d6288/vpc.tf)

``` HCL
provider "aws" {
  region = "eu-west-1"
  access_key = "ACCESS_KEY_HERE"
  secret_key = "SECRET_KEY_HERE"
}

resource "aws_vpc" "vpc" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "DemoVPC"
  }
}
```
Terraform uses HCL, or Hashicorp Configuration Language for it's definition files, which are quite readable (you can also use JSON if you wish).
The provider sections tells Terraform that we are going to be working with AWS. First is the region that resources will be created in. I'm using eu-west-1 but feel free to change that to whatever suits. Next up is the access and secret keys that will be used for authentication. As a rule you should never put these in any files that will be checked into source control and Terraform supports alternative ways of supplying these values, which are listed in the Terraform documentation. If you have the AWS CLI installed your can type `AWS configure` at the command line and you'll be prompted to provide your access keys and a region, which will become the defaults for AWS related activity. Terraform will also use these values if present so if you set these up you can remove the values from the provider configuration.
The next section defines our vpc resource; a resource is anything that will be created as a result of our Terraform operation. The general format for defining resources is `resource resource_type variable_name`. The allowed values for resource type is determined by the type of provider being used. The variable name is like a variable in any programming language, it's name has no effect on the end result but it allows us to refer to this resource in other places, as we'll see.
We set some options for our VPC (see [this post](http://pdsutcliffe.co.uk/2018-09-03/cidr-blocks) about Cidr blocks for more explanation on this topic) and add a tag (most resources allow tags to be added).

The first time we work with a terraform configuration, we need to initialize our work space by running `terraform init` from the commandline, which will download the appropriate plugins for the configuration we're using. Once that's done `terraform plan` will show us what Terraform will do when we apply our configuration. In this case we should see a list of details followed by

```
Plan: 1 to add, 0 to change, 0 to destroy.
```
Terraform will remember the actions that have been applied in a state file and can make updates in an additive manner. Our tf configuration files should always reflect the final state that we desire our resources to be. If we change or remove the resources defined in these files, Terraform will add / modify or remove resources as necessary to achieve the desired state. Pay attention to the output of Terraform plan to ensure you don't get any nasty surprises.

One more thing before we run the plan: it's often useful to get feedback on the resources that Terraform has created such as ip addresses of servers or DNS names of load balancers. Let's ask Terraform to tell us the id of the VPC it creates. Add a new file [**outputs.tf**](https://raw.githubusercontent.com/PeteSutcliffe/aws-vpc-terraform/b7358083734b2c1e236d62025ac980ca7c0d6288/outputs.tf) with contents

``` HCL
output "vpc_id" {
  value = "${aws_vpc.vpc.id}"
}
```

Here we can see Terraform's interpolation syntax in action. We're asking it to output a value labelled "vpc_id" the value of which is going to come from the vpc variable we defined. The interpolation syntax is `${resource_type.variable_name.attribute}`

Run `terraform apply` from the command line and if all goes well you should see a success message:

```
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

vpc_id = vpc-07568f4db01a18aaf
```

Go check in the AWS console and you should be able to find the VPC listed in the correct region:

![vpc_id.PNG]({{site.baseurl}}/images/vpc_id.PNG)

Finally, to destroy all resources to reduce costs run `terraform destroy` you'll be prompted to enter "yes" to confirm then everything will be removed. There isn't any cost just for creating a VPC so it's not really necessary at this stage but now we're automating our infrastructure we can easily recreate everything when we need it so there's no point getting too attached to things.

### Summary

We've learned some basics about Terraform and created a VPC that will contain all our resources. Next time we'll look at creating some servers to play with.
