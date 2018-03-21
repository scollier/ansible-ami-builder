# Getting started with AAB

## General Overview

AAB builds AMIs in 4 steps

* Provisions a instance on AWS using the RHEL base AMI ID that you specify.  Everything starts from there.
* AAB then registers that instance with route53 so you can ssh into it via public hostname.
* Next AAB configures the instance using playbooks that are called by roles that you specify. This is completely customizable.
* Finally, AAB creates an AMI from that image that you can use for other projects

## Prerequisites

* Configured AWS VPC, routes, security groups, private key
* boto
* AWS account with keys to make API calls
* AWS private key to log into instances
* RHN username and password with access to the proper channels
 * List of pools that have access to the proper channels
* secrets file to store keys, 
* RHEL base AMI ID

## Building an AMI with AAB

1. First you need to clone this repository

```git clone https://github.com/scollier/ansible-ami-build.git```

2. Put the AWS private key into your keyring.

```eval `ssh-agent` && ssh-add ./aws-private-key.pem && ssh-add -l```

3. Configure your `secrets.yml` file with your environment configuration and the profiles you want to use. There is an example in the root directory of this repository. The secrets file is properly commented to provide some guidance on what is required.

4. Run AAB and create your AMI.

```ansible-playbook -vvv -e @scollier_secrets-ami_build.yml ami_lab_launch.yml```
