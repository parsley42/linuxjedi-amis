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
* A permissive .ssh/config that doesn't sweat changing host keys, e.g.:
```
CanonicalizeHostname yes
CanonicalDomains linuxjedi.org
Host *.linuxjedi.org
        StrictHostKeyChecking false
        UserKnownHostsFile /dev/null
        CheckHostIP false
```

## Creating an Image Manually

1. Create an instance with the same name as the template:
```bash
$ ec2 create f28img c9-native f28img
info: Loading configuration and variables from ../linuxjedi-private/cloudconfig.yaml
info: Loading configuration and variables from ../aws-linuxjedi/cloudconfig.yaml
info: Loading configuration and variables from ./cloudconfig.yaml
info: Loading api template /home/ec2-user/environment/ruby-awstools/lib/rawstools/templates/ec2/ec2.yaml
info: Loading api template ./ec2/f28img.yaml
201808020623 Created instance f28img (id: i-026b9eb937050acd4), waiting for it to enter state running ...
201808020623 Running
201808020623 Adding public DNS record f28img -> 35.172.233.207
info: Loading api template /home/ec2-user/environment/ruby-awstools/lib/rawstools/templates/route53/arec.yaml
201808020623 Adding private DNS record f28img -> 10.42.0.205
info: Loading api template /home/ec2-user/environment/ruby-awstools/lib/rawstools/templates/route53/arec.yaml
```
2. Run the `img_build` playbook:
```bash
$ ansible-playbook img_build.yaml -e target=f28img
```
3. Stop the instance:
```bash
$ ec2 stop f28img
```
4. Get the instance ID:
```bash
$ ec2 list instances | grep ^f28img | (read IMG STATE INST REST; echo $INST)             
info: Loading configuration and variables from ../linuxjedi-private/cloudconfig.yaml
info: Loading configuration and variables from ../aws-linuxjedi/cloudconfig.yaml
info: Loading configuration and variables from ./cloudconfig.yaml
i-05a070d60e2f67032
```
5. Create the AMI:
```bash
$ aws ec2 create-image --instance-id i-05a070d60e2f67032 --name f28base.20180802
{
    "ImageId": "ami-09f208827020cb047"
}
```
6. Store the new AMI ID in parameter store:
```bash
$ param store /ami/f28base ami-09f208827020cb047
...
Stored parameter /ami/f28base = ami-09f208827020cb047
```