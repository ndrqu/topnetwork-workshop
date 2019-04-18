# Packer build AMI images

First of all, install Packer following the vendos instruction:

https://www.packer.io/intro/getting-started/install.html#precompiled-binaries

Set your AWS credentials as environment variables:

```bash
export AWS_ACCESS_KEY_ID=MYACCESSKEYID
export AWS_SECRET_ACCESS_KEY=MYSECRETACCESSKEY
```

Now starting from an Ubuntu 18.04LTS AMI images you configure Packer in order to install latest NGINX web server.

Create a bash script, called install--nginx.sh to setup the VM:

```bash
#!/bin/bash
sudo apt-get -y update
sudo apt-get -y install nginx
sudo systemctl enable nginx
sudo systemctl disable ufw
```

Create packer.json file, add teamID to "ami_name" parameter:

```json
{
    "variables": {
        "aws_access_key": "{{env `AWS_ACCESS_KEY_ID`}}",
        "aws_secret_key": "{{env `AWS_SECRET_ACCESS_KEY`}}",
        "region":         "eu-west-1"
    },
    "builders": [
        {
            "access_key": "{{user `aws_access_key`}}",
            "ami_name": "packer-linux-aws-demo-{{timestamp}}",
            "instance_type": "t2.micro",
            "region": "eu-west-1",
            "secret_key": "{{user `aws_secret_key`}}",
            "source_ami_filter": {
              "filters": {
              "virtualization-type": "hvm",
              "name": "ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*",
              "root-device-type": "ebs"
              },
              "owners": ["099720109477"],
              "most_recent": true
            },
            "ssh_username": "ubuntu",
            "type": "amazon-ebs"
        }
    ],
    "provisioners": [
        {
            "type": "shell",
            "script": "./install--nginx.sh"
        }
    ]
}
```

There are three blocks inside the Packer manifest:

* variables: include the credential in order to authenticate Packer to setup and upload the AMI images
* builders: a set of options to select the base image and the output AMI name
* provisioner: a set of instruction to be executed in order to customize the AMI

Let's validate the template by running

```bash
packer validate packer.json
```

The output should look similar to:

```bash
Template validated successfully.
```

And to build, run:

```bash
packer build packer.json
```

Your output will look like this:

```bash
amazon-ebs output will be in this color.

==> amazon-ebs: Prevalidating AMI Name: packer-linux-aws-demo-1555421594
    amazon-ebs: Found Image ID: ami-0727f3c2d4b0226d5
==> amazon-ebs: Creating temporary keypair: packer_5cb5d99a-66b5-ef19-219b-ce30b20407ed
==> amazon-ebs: Creating temporary security group for this instance: packer_5cb5d99c-839c-3f2b-7ae5-767babda657e
==> amazon-ebs: Authorizing access to port 22 from 0.0.0.0/0 in the temporary security group...
==> amazon-ebs: Launching a source AWS instance...
==> amazon-ebs: Adding tags to source instance
    amazon-ebs: Adding tag: "Name": "Packer Builder"
    amazon-ebs: Instance ID: i-06fe22d843a16d6ab
==> amazon-ebs: Waiting for instance (i-06fe22d843a16d6ab) to become ready...
==> amazon-ebs: Using ssh communicator to connect: 52.209.23.82
==> amazon-ebs: Waiting for SSH to become available...
==> amazon-ebs: Connected to SSH!

OTHER STUFF

Build 'amazon-ebs' finished.

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
eu-west-1: ami-08cd8be3044ec189e
```

The last row show the AMI ID.
Connect to the AWS console: Service -> EC2 -> Images -> AMIs and search for the AMI ID created by Packer.
Your AMI ID will be different.

## Terraform

For this tutorial we working with Terraform Configuration Language 0.11.
Terraform code is written in the HashiCorp Configuration Language (HCL) in files with the extension .tf.
It is a declarative language, so your goal is to describe the infrastructure you want, and Terraform will figure out how to
create it. 

The first step to using Terraform is typically to configure the provider(s) you want to use. Create an empty folder and
put a file in called main.tf with the following contents: 

```bash
provider "aws" { 
    region = "eu-west-1" 
} 
```

This tells Terraform that you are going to be using AWS as your provider and that you wish to deploy your
infrastructure into the eu-west-1 region, located in Ireland. 

Remember to don’t violate Repeat Yourself (DRY) principle: every piece of knowledge must have a single,
unambiguous, authoritative representation within a system. Let's use variables.

Create variable.tf file:

```bash
variable "aws_region" {
    description = "AWS region"
    default = "eu-west-1" 
}

variable "public_key_path" {
    description = "SSH key used for EC2 access"
}

variable "key_name" {
    description = "SSH key name"
}

variable "aws_amis" {
    description = "Put the AMI ID created by Packer"
    default = {
        eu-west-1 = "ami-08cd8be3044ec189e"
    }
}

variable "teamid" {
    description = "Put the TeamID"
    default = 
}
```


## Authentication

Provide your credentials via the AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, environment variable:

```bash
export AWS_ACCESS_KEY_ID="anaccesskey"
export AWS_SECRET_ACCESS_KEY="asecretkey"
```

## VPC

VPC , or Virtual Private Cloud, is an isolated area of your AWS account that has its own virtual network and IP address space. 

Create a file called network.tf and put the network configuration:

```bash
# Create a VPC to launch our instances into
resource "aws_vpc" "top_networks_vpc" {
  cidr_block = "172.${var.teamid}.0.0/16"
}

# Create an internet gateway to give our subnet access to the outside world
resource "aws_internet_gateway" "top_networks_ig" {
  vpc_id = "${aws_vpc.top_networks_vpc.id}"
}

# Grant the VPC internet access on its main route table
resource "aws_route" "top_networks_internet_access" {
  route_table_id         = "${aws_vpc.top_networks_vpc.main_route_table_id}"
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = "${aws_internet_gateway.top_networks_ig.id}"
}

# Create a subnet to launch our instances into
resource "aws_subnet" "top_networks_subnet_web1a" {
  vpc_id                  = "${aws_vpc.top_networks_vpc.id}"
  cidr_block              = "172.${var.teamid}.0.0/28"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1a"
}

resource "aws_subnet" "top_networks_subnet_web1b" {
  vpc_id                  = "${aws_vpc.top_networks_vpc.id}"
  cidr_block              = "172.${var.teamid}.16.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1b"
}

resource "aws_subnet" "top_networks_subnet_ddb1a" {
  vpc_id                  = "${aws_vpc.top_networks_vpc.id}"
  cidr_block              = "172.${var.teamid}.208.0/28"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1a"
}

resource "aws_subnet" "top_networks_subnet_ddb1b" {
  vpc_id                  = "${aws_vpc.top_networks_vpc.id}"
  cidr_block              = "172.${var.teamid}.224.0/28"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1b"
}

resource "aws_subnet" "top_networks_subnet_elb1a" {
  vpc_id                  = "${aws_vpc.top_networks_vpc.id}"
  cidr_block              = "172.${var.teamid}.32.0/28"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1a"
}

resource "aws_subnet" "top_networks_subnet_elb1b" {
  vpc_id                  = "${aws_vpc.top_networks_vpc.id}"
  cidr_block              = "172.${var.teamid}.48.0/28"
  map_public_ip_on_launch = true
  availability_zone       = "eu-west-1b"
}
```

In a terminal, go into the folder where you created files, and run the terraform init command: 

```bash
terraform init
```

This command install modules dependencies in order to convert your terraform plan and execute it on AWS.
After this command you're ready to "plan" your infrastructure!
Note, if variables are defined but not valorized, before plan terraform pront to insert a value:


```bash
terraform plan -out=out.tfplan 
```

The plan command lets you see what Terraform will do before actually making any changes. This is a great way to
sanity check your code before unleashing it onto the world. The output of the plan command is similar to the output
of the diff command that is part of Unix, Linux, and git resources.
Sign (+) are going to be created, resources with a minus sign (–) are going to be deleted, and resources with a tilde sign (~) are going to be modified.

A state of the plan will be saved on out.tfplan file, checkout the content!
If the output of the plann look good and not try to delete production resource, appluy it!

```bash
terraform apply "out.tfplan"
```


## Security Group

By default, AWS does not allow any incoming or outgoing traffic from an EC2 or database Instance. 
To allow AWS services to receive traffic from internet or other subnet, you need to create a Security Group a.k.a. firewall.

Add to network.tf:

```bash
#--------------------------------SECURITY GROUP
# A security group for the ELB so it is accessible via the web
resource "aws_security_group" "top_networks_elb_sg" {
  name        = "top_networks_elb_sg_${var.teamid}"
  description = "never safe enought"
  vpc_id      = "${aws_vpc.top_networks_vpc.id}"

  # HTTP access from anywhere
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # outbound internet access
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Our default security group to access
# the instances over SSH and HTTP
resource "aws_security_group" "top_networks_default_sg" {
  name        = "top_networks_elb_default_${var.teamid}"
  description = "never safe enought"
  vpc_id      = "${aws_vpc.top_networks_vpc.id}"

  # SSH access from anywhere
  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # HTTP access from the VPC
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["172.0.0.0/16"]
  }

  # outbound internet access
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Now plan and if is SAFE execute!

## EC2

In AWS a server or virtual machine is known as an EC2 Instance.

To deploy an EC2 instance, firt upload a SSH key, to allow user authentication.
Add in ec2.tf:

```bash
resource "aws_key_pair" "top_networks_auth" {
  key_name   = "${var.key_name}"
  public_key = "${var.public_key_path}"
}
```

Now add the configuration for the EC2 provisioning:

```bash
resource "aws_instance" "top_networks_ec2" {
  connection {
    # The default username for our AMI
    user = "ubuntu"
  }

  # instance size
  instance_type = "t2.medium"

  # Lookup the correct AMI based on the region
  # we specified
  ami = "${lookup(var.aws_amis, var.aws_region)}"

  # The name of our SSH keypair we created above.
  key_name = "${aws_key_pair.top_networks_auth.id}"

  # Our Security group to allow HTTP and SSH access
  vpc_security_group_ids = ["${aws_security_group.top_networks_default_sg.id}"]

  # We're going to launch into the same subnet as our ELB. In a production
  # environment it's more common to have a separate private subnet for
  # backend instances.
  subnet_id = "${aws_subnet.top_networks_subnet_web1a.id}"

  tags {
    Name   = "nginx-webserver"
    teamid = "${var.teamid}"
  }
}
```
If everithings will be fine, last row of the output will be:

```bash
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

Try to connect to the EC2! Retrive public IP address on the AWS console (pay attention on teamid tag!). 

## Elastic Load Balancer

A load balancer distribute traffic across your servers.
Elastic Load Balancer is highly available and scalable.

To create an ELB with Terraform, you use the aws_elb resource: 
This creates an ELB that will work across all of the AZs in your account. 

```bash
resource "aws_elb" "top_networks_elb" {
  name = "topnetworkselb-${var.teamid}"

  subnets         = ["${aws_subnet.top_networks_subnet_elb1a.id}", "${aws_subnet.top_networks_subnet_elb1b.id}"]
  security_groups = ["${aws_security_group.top_networks_elb_sg.id}"]
  instances       = ["${aws_instance.top_networks_ec2.id}"]

  listener {
    instance_port     = 80
    instance_protocol = "http"
    lb_port           = 80
    lb_protocol       = "http"
  }
}
```

before provisioning the ELB creare output.tf file and add:

```bash
output "address" {
  value = "${aws_elb.top_networks_elb.dns_name}"
}
```

Plan and execute. Look the final output, will be a little different:

```bash
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

address = topnetworkselb-0-952846767.eu-west-1.elb.amazonaws.com
```

Terraform wrote the ELB DNS name, so copy and paste it in a web browser:

```bash
curl -v  topnetworkselb-0-2056735367.eu-west-1.elb.amazonaws.com
* Rebuilt URL to: topnetworkselb-0-2056735367.eu-west-1.elb.amazonaws.com/
*   Trying 52.19.101.155...
* TCP_NODELAY set
* Connected to topnetworkselb-0-2056735367.eu-west-1.elb.amazonaws.com (127.0.0.1) port 80 (#0)
> GET / HTTP/1.1
> Host: topnetworkselb-0-2056735367.eu-west-1.elb.amazonaws.com
> User-Agent: curl/7.54.0
> Accept: */*
> 
< HTTP/1.1 200 OK
< Accept-Ranges: bytes
< Content-Type: text/html
< Date: Thu, 18 Apr 2019 05:03:50 GMT
< ETag: "5cb7fba0-264"
< Last-Modified: Thu, 18 Apr 2019 04:22:56 GMT
< Server: nginx/1.14.0 (Ubuntu)
< Content-Length: 612
< Connection: keep-alive
< 
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
* Connection #0 to host topnetworkselb-0-2056735367.eu-west-1.elb.amazonaws.com left intact
```

NOTE it may take some time before the instances will be online.