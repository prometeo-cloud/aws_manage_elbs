# aws_manage_elbs

## Description:

Manages AWS Elastic Load Balancers (ELBs) as follows:

- Running the role will create the ELBs and any associated Target Groups defined in `aws_elbs` when the variable `arg_action` is not defined or is defined and set to `create`
- Running the role will delete the ELBs and any associated Target Groups defined in `aws_elbs` when the variable `arg_action` is defined and is set to `delete`

For Ansible 2.5 or greater, this role requires Boto3 to be installed as follows:

`sudo pip install boto3`

or:

```yaml
- name: Install boto3
  easy_install:
    name: boto3
    state: present
```

## Dependencies

This role uses the AWS cli.  It therefore requires AWS credentials as environment variables or in a configuration file (see [here](https://docs.aws.amazon.com/cli/latest/topic/config-vars.html#cli-aws-help-config-vars)).  The role `aws_credentials` will create a config file for use with the AWS cli.

## Behaviour:

**Feature:** Create AWS ELBs and associated Target Groups  
- **Given** valid AWS credentials
- **Given** existing VPCs
- **When** the script is executed when `arg_action` is not defined or is defined and not set to `delete`
- **Then** the ELBs and associated Target Groups are created
- **When** the script is executed when `arg_action` is defined and is set to `delete`
- **Then** the ELBs and associated Target Groups are deleted

## Configuration:

Common variables used by this role:

| Variable | Description | Example |
|-----|-----|-----|
| **aws_region** | AWS region | See [AWS regions](http://docs.aws.amazon.com/general/latest/gr/rande.html#ec2_region) |
| **aws_access_key** | AWS access key | AKIAIOSFODNN7EXAMPLE |
| **aws_secret_key** | AWS secret key | wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY |
| **resource_tags** | List of resource tags | see usage |
| **master_cert** | Master Certificate | `arn:partition:service:region:account-id:resourcetype/resource` | group_vars/all/vars |
| **apps_router_cert** | Application/Router Certificate | `arn:partition:service:region:account-id:resourcetype/resource` | group_vars/all/vars |

Variables specific to this role:

| Variable | Description | Example |
|-----|-----|-----|
| **aws_elbs** | List of Target Groups to create. | see usage |

## Usage:

How to invoke the role from a playbook (create ELBs and Target Groups):

```yaml
- name: Create AWS ELBs and Target Groups
  include_role:
    name: "aws_manage_elbs"
  vars:
    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London
    resource_tags:
      Name: "{{ resource_name }}"   # Set to upper case aws_elbs.name
      Owner: "Acme Co."
    master_cert: "arn:partition:service:region:account-id:resourcetype/resource"
    apps_router_cert: "arn:partition:service:region:account-id:resourcetype/resource"

    aws_elbs:
      - name: "test-master-lb-ext"
        region: "{{ aws_region }}"
        scheme: "internet-facing"
        subnets:
          - "TEST-Infra-Public-A"
          - "TEST-Infra-Public-B"
          - "TEST-Infra-Public-C"
        target_group:
            name: "test-master-ext-tg"
            protocol: "tcp"
            port: 80
        type: "network"
        vpc_name: "Test Infra VPC"
    
      - name: "test-master-lb-int"
        region: "{{ aws_region }}"
        scheme: "internal"
        subnets:
          - "TEST-Infra-Private-A"
          - "TEST-Infra-Private-B"
          - "TEST-Infra-Private-C"
        target_group:
          name: "test-master-int-tg"
          protocol: "tcp"
          port: 80
        type: "network"
        vpc_name: "Test Infra VPC"
```

How to invoke the role from a playbook (delete ELBs and Target Groups):

```yaml
- name: Delete AWS ELB Target Groups
  include_role:
    name: "aws_elb_target_group_manage"
  vars:
    arg_action: "delete"

    aws_access_key: AKIAIOSFODNN7EXAMPLE
    aws_secret_key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
    aws_region: eu-west-2  # London

    aws_elbs:
      - name: "test-master-lb-ext"
        region: "{{ aws_region }}"
        target_group:
            name: "test-master-ext-tg"
    
      - name: "test-master-lb-int"
        region: "{{ aws_region }}"
        target_group:
          name: "test-master-int-tg"
```
