# `aws-ssh`

## Introduction

Having active AWS credentials, show and execute the `ssh` commandline to connect
to:

* Bastion host
* ECS host
* RDS database instance

## How it works

* Find the IP address of a EC2 instance with a tag `bastion`
* Find the IP address of EC2 instances in the private networks
* Get the `ssh` key name assigned to the `bastion` host
* Show the `ssh` command to set up the tunnel or to log in to the
  host
  
## Prerequisites

* [`aws-sts-assumerole`](https://github.com/rik2803/aws-sts-assumerole)
* Active credentials for the account you want to interact with. These
  credentials can be set with previous dependency, `aws-sts-assumerole`.
* A valid private SSH key for the account. The SSH key should have the
  name as the key name in AWS, and should be available in the directory
  `~/.ssh` (default) or the directory set in the environment variable
  `${PRIVKEYHOME}`. The private key will be loaded with `ssh-add` if not
  already loaded.
* If access to your AWS SSH assets are protected by a VPN, the VPN should
  be connected in order to be able to `ssh` in to the resources in that
  account.
  
## Subcommands

### `bastion`

Start a `ssh` session on the _Bastion_ host.

```
$ ~/projects/AWS/aws-ssh/aws-ssh bastion
INFO - Private key id_rsa_XXXXXXXX already loaded
INFO - Log in to bastion account
Last login: Thu Oct  4 06:03:16 2018 from host.acme.com

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
27 package(s) needed for security, out of 374 available
Run "sudo yum update" to apply all updates.
```

You can optionally add a command to execute on the remote host:

```
$ aws-ssh bastion sudo ip a
INFO - Private key id_rsa_ixor.ixordocs-prd already loaded
INFO - Log in to bastion account
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
...
Connection to 1.2.3.4 closed.
[~][aws: ixordocs-prd]
```

### `ecs`

When running `aws-ssh ecs`, the `ssh` commands to connect to the ECS hosts
on the account for which credentials are active will be printed to `stdout`.

```
$ ./aws-ssh ecs
INFO - Private key id_rsa_lemo-acc already loaded
INFO - Show commands to connect to ec2 instances
ssh -t -A ec2-user@1.2.3.4 'ssh -A 5.6.7.8'
ssh -t -A ec2-user@1.2.3.4 'ssh -A 9.10.11.12'
```

When only 1 ECS host was found, the `ssh` command will be executed rather than
printed on `stdout`.

You can optionally add a command to execute on the remote host:

```bash
$ aws-ssh ecs docker stats
$ aws-ssh ecs docker info
INFO - Private key id_rsa_ixor.ixordocs-prd already loaded
INFO - Show commands to connect to ECS ec2 instances
Containers: 8
 Running: 8
 Paused: 0
 Stopped: 0
Images: 9
...
```


### `dockerps`

Show all _Docker_ containers running on the AWS account. Useful if you want to connect
to the ECS host where a certain container is running.

```
$ ./aws-ssh dockerps
INFO - Private key id_rsa_XXXXXXX already loaded
INFO - Show docker container info on all ECS hosts
INFO - Show docker containers on 5.6.7.8
CONTAINER ID        IMAGE
60663c70bfce        123456789012.dkr.ecr.eu-central-1.amazonaws.com/XXXX/image001:latest
cb5a0595b9d6        123456789012.dkr.ecr.eu-central-1.amazonaws.com/XXXX/image002:latest
7998df9e8008        123456789012.dkr.ecr.eu-central-1.amazonaws.com/XXXX/image003:latest
efecf61ca32d        riktytgat/https-redirect-with-healthcheck
1d72eb548a79        amazon/amazon-ecs-agent:latest
Connection to 5.6.7.8 closed.
INFO - Show docker containers on 9.10.11.12
CONTAINER ID        IMAGE
f33d8e093aee        123456789012.dkr.ecr.eu-central-1.amazonaws.com/XXXX/image001:latest
d0be09bfe23a        amazon/amazon-ecs-agent:latest
Connection to 9.10.11.12 closed.
```

### `rdstunnel`

Prints the commands to set up a tunnel to the RDS databases on the account.

```
$ ./aws-ssh rdstunnel
INFO - Private key id_rsa_XXXXXXX already loaded
INFO - Show command to setup tunnel to RDS
ssh -f -Nnt -A -L 3306:mysql-01.jdien58dhh59d.eu-central-1.rds.amazonaws.com:3306 ec2-user@1.2.3.4
```


## Troubleshooting

### Too many authentication failures

```
$ ssh -t -A ec2-user@18.197.51.77 'ssh -A 1.2.3.4'
Received disconnect from 10.120.10.197 port 22:2: Too many authentication failures
Authentication failed.
Connection to 18.197.51.77 closed.
```

This usually means that you have too many keys (>5) loaded in yous SSH agent:

```
$ ssh-add -l | wc -l
       7
```

Remove the keys you will not be using soon until you hav 5 or less:

```
$ ssh-add -d <full-path-to-private-key>
```
