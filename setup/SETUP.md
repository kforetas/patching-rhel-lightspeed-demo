# Set up the demo environment

## Environment to be set up
- Single VPC
- Single subnet
- Single route table
- Single gateway
- Single managed server for dev and prod respectively
- Single Ansible Automation Platform

The setup looks like the following:
![](../images/patching-rhel-lightspeed-demo.png)


## Included contents
### Roles
|Name     |Description|
|:--------|:----------|
|managed  |A role to create managed servers registered to Red Hat Lightspeed.|
|aap      |A role to create an Ansible Automation Platform on AWS EC2.|

### Playbooks
|Name     |Role Used|Description|
|:--------|:--------|:----------|
|`create_networks.yml`|N/A|Create required AWS network resources.|
|`delete_networks.yml`|N/A|Delete AWS network resources created in `create_networks` playbook.|
|`create_managed_vms.yml`|[roles.managed](roles/managed/README.md)|Create AWS instances and set up managed servers.|
|`delete_managed_vms.yml`|N/A|Delete the instances created in `create_managed_vms` playbook.|
|`create_aap_vm.yml`|[roles.aap](roles/aap/README.md)|Create an AWS instance and set up Ansible Automation Platform.|
|`delete_aap_vm.yml`|N/A|Delete the instance created in `create_aap_vm` playbook.|


## Prerequisites
### Basic requirements for Ansible
Any control node:
- ansible core 2.18+

Ansible collections:
- amazon.aws
- community.crypto
- community.mysql
- redhat.rhel_system_roles
- redhat.insights

In order for functioning Ansible EC2, you need to install Python Boto3 library.
```
# pip3 install boto3
```
### ansible.cfg
Update the following line with your EC2 private key file.
```
[default]
private_key_file = "path to EC2 private key file"
```
### Environment variables
In this setup, you should set the following environment variables on your control node.
```
$ export AWS_DEFAULT_REGION=ap-northeast-1
$ export AWS_ACCESS_KEY_ID=AKIAIOSFODNN7EXAMPLE
$ export AWS_SECRET_ACCESS_KEY=wJatrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```

### Subscriptions
Basically, the demo environment requires to get access to Red Hat gold image, so you must have a matching Red Hat product subscription and must connect their cloud provider accounts to Red Hat.
Please refer to the [Cloud Access user interface](https://access.redhat.com/management/cloud) or Cloud Sources on [cloud.redhat.com](https://cloud.redhat.com/) as described in [Red Hat Cloud Access program overview](https://docs.redhat.com/en/documentation/subscription_central/1-latest/html/getting_started_with_rhel_system_registration/con-red-hat-cloud-access-program-overview).

Required subscriptions:
- Red Hat Enterprise Linux for x86_64
- Red Hat Ansible Automation Platform Subscription

## Usage
### Create required network resources
These variables should be set in group_vars beforehand.
```
aws_vpc: patch_demo_vpc
aws_vpc_cidr_block: 10.1.0.0/16 # adjust with your preference
aws_vpc_subnet_name: patch_demo_subnet
aws_subnet_cidr_block: 10.1.1.0/24 # adjust with your preference
aws_igw_name: patch_demo_gtw
aws_routetable_name: patch_demo_rtb
aws_securitygroup_name: patch_demo_sg

purpose: patch_demo
```

This playbook need to be run at the beginning.
```
$ ansible-playbook create_networks.yml
```

### Create managed servers
These variables should be set in group_vars beforehand.
```
aws_vpc_subnet_name: patch_demo_subnet
aws_securitygroup_name: patch_demo_sg
aws_managed_instance:
  dev:
    ami: ami-06b08b819edcd2cf0 # ami of RHEL-9.7.0_HVM-20260513-x86_64-0-Hourly2-GP3
    size: t2.small # can be bigger instance size
  prod:
    ami: ami-0dc4c409764b96fa9 # ami of RHEL-9.7.0_HVM-20260120-x86_64-0-Hourly2-GP3
    size: t2.small # can be bigger instance size

managed_vms_type: managed # should not be modified
managed_vms_name_prefix: managed # should not be modified
managed_vms_environment:
  - dev # should not be modified
  - prod # should not be modified

purpose: patch_demo
```

And, the following variables are prompted at run-time.  Also refer to [roles.managed](roles/managed/README.md) for the role details.
```
aws_keypair_name # Your AWS key pair name corresponding to the private key
rhsm_username # Red Hat Login name
rhsm_passwd # Password for Red Hat Login
mysql_root_passwd # MySQL root password for WordPress
mysql_wp_passwd # MySQL user password for WordPress
```

This playbook can run after running `create_networks` playbook.
```
$ ansible-playbook create_managed_vms.yml
```

### Create Ansible Automation Platform
These variables should be set in group_vars beforehand.
```
aws_vpc_subnet_name: patch_demo_subnet
aws_securitygroup_name: patch_demo_sg
aws_aap_instance_ami: ami-07fe9ea188b04b2e6 # ami of RHEL-9.8.0_HVM-20260618-x86_64-0-Hourly2-GP3
aws_aap_instance_size: t2.xlarge # should not be modified

aap_vm_type: aap # should not be modified
aap_vm_name: aap01

purpose: patch_demo
```

And, the following variables are prompted at run-time. Also refer to [roles.aap](roles/aap/README.md) for the role details.
```
aws_keypair_name # Your AWS key pair name corresponding to the private key
rhsm_username # Your Red Hat login name
rhsm_passwd # Password for your Red Hat login
aap_admin_passwd # Password for your AAP admin user
aap_pg_passwd # PostgreSQL password for your AAP deployment
```

This playbook can run after running `create_networks` playbook.
```
$ ansible-playbook create_aap_vm.yml
```

### Clean up the environment
All the delete resource playbooks corresponding to each create resource playbook are avaialble. Those playbooks can run assuming related variables have already set previously.
