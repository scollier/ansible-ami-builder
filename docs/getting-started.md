# Getting started with AAB

## General Overview

AAB builds AMIs in 4 steps using a 'profile' which specifies custom tasks to perform.

1. Provisions a instance on AWS using the RHEL base AMI ID and size that you specify.  Everything starts from there.
1. AAB then registers that instance with route53 so you can ssh into it via public hostname.
1. Next AAB configures the instance using a two sets of playbooks that are 1. common to all profiles and 2. customized for the specific profile. These is completely customizable.
1. Finally, AAB creates an AMI from that image that you can use for other projects (this can be toggled off if desired)

## Prerequisites

* Ansible >= 2.4
* Configured AWS VPC, routes, security groups, private key
* boto on the localhost that executes these playbooks
* AWS account with keys to make API calls
* AWS private key to log into instances
* RHN username and password with access to the proper channels
* List of pools that have access to the proper channels
* secrets file to store keys, passwords, etc.
* RHEL base AMI ID

## Building an AMI with AAB for pre-configured profiles

1. First you need to clone this repository

```git clone https://github.com/scollier/ansible-ami-build.git```

2. Put the AWS private key into your keyring.

```eval `ssh-agent` && ssh-add ./aws-private-key.pem && ssh-add -l```

3. Configure your `secrets.yml` file with your environment configuration and the profile you want to use via the 'ami_config' variable. As of this writing the supported profiles are one of ['http','ocp','tower','rh-container-lab','ansible_host'] but others can be added, see *Add Custom Profiles* section below. There is an example `secrets.yml` file in the root directory of this repository and is properly commented to provide some guidance on what is required.

4. Run AAB and specify which profile to build and create your AMI. A

```ansible-playbook -vvv -e @scollier_secrets-ami_build.yml ami_lab_launch.yml```

## Tweaking Advanced Options

Instead of specifying all options in an included yaml file like `secrets.yml` you can instead override specific variables on the command-line when calling `ansible-playook` Examples would be for seting the profile with `-e ami_config=tower` or perhaps when creating a profile for `ansible_host` and skipping the AMI creation with `-e ami_config=tower -e ami_create=False`.

## Adding Custom profiles
AAB is designed to apply custom profile tasks based on the variable 'ami_config'. It is trivial to add new profiles by doing the following. This example will create a custom profile called 'foo'

1. Create a file in the `roles/ami_config/tasks/profiles/` directory called 'foo.yml'. Alternatively, you may create a subdirectory called `foo` and `foo.yml` underneath it if there are complex or multiple tasks that need to be applied to this profile. See the 'tower' profile as an example.
1. Create a custom variable file for this profile and place it in `roles/ami_config/vars/foo.yml`. Be sure to refer to one of the other profile variable files in this directory to understand which variables to specify, as some are required. For example, every profile should define `yum_repos`, `yum_packages`, `pip_packages` as well as booleans `yum_update`, `epel_setup`, and `ami_create`. Add any profile-speific variables here.
1. Edit `roles/ami_config/tasks/profiles/vars.yml` and follow the existing pattern to import your profile var file from `roles/ami_config/vars/foo.yml`
1. Edit `roles/ami_config/tasks/profiles/tasks.yml` and follow the existing pattern to import your profile task file from `roles/ami_config/tasks/profiles/foo.yml`
