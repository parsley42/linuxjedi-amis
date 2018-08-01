This repository contains configuration and data for creating AWS AMIs for use at LinuxJedi.org. The goal is to create a set of normalized AMIs manageable with Ansible:
* Standard 'build' user (not fedora/ubuntu/ec2-user) with homedir: /build
* Python installed

Additionally, specifically for LinuxJedi.org:
* Install latest "Go"
* Install `ruby-awstools`
* Packages supporting Cloud9 (nodejs, glibc-static, etc.)
* The aws cli

Requirements:
* aws-linuxjedi - repository used for configuring the VPC and other CloudFormation resources
* linuxjed-private - repository with non-public variable definitions
* AMI IDs stored in SSM Parameter Store:
  * /ami/amzn2
  * /ami/f28
  * /ami/u16
