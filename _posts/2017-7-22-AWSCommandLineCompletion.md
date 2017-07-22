---
layout: post
title: AWS command line completion
tags: aws cli bash completion
---


Amazon Web Services or AWS has a terrific web console, but sometimes I just like to use the command line.  It is particularly useful to do something programmatic like listing your instance PublicDNSName.  An instance is just a server that is running in AWS and the PublicDNSName is just the name you need to connect, typically that connection is done via ssh from the command line. 
```bash
hn=$(aws ec2 describe-instances --instance-ids=i-0bd7d505ae0449cd2 | awk '/PublicDnsName/ {print $2 }' | sort -u | tr -d , | tr -d \")
```

I've also been enjoying using the AWS cli for small tasks like adding firewall rules to security zones and similar.
```bash
# get myip to add to the access rules
myip=$(dig +short myip.opendns.com @resolver1.opendns.com)
echo using $myip for ssh source address

# TODO add security group lookup, but I bet most new people only have 1.
sgname=my-sg-name-here

set +e
aws ec2 describe-security-groups --group-names $sgname --filter="Name=ip-permission.cidr,Values=${myip}/32,Name=ip-permission.from-port,Values=22" | grep -q $myip
RC=$?
set -e
if [[ $RC -ne 0 ]]; then
        # not there, so add it.
        aws ec2 authorize-security-group-ingress --group-name $sgname --protocol tcp --port 22 --cidr ${myip}/32
fi
```

OK so that is great, but even after you use the command for awhile remembering some of the names (was it aws ec2 start-instance or aws ec2 start-instances?) takes time.  Sure enough they offer command line completion and the [manual](http://docs.aws.amazon.com/cli/latest/userguide/cli-command-completion.html) shows how. For me it was just adding a line to my ~/.bashrc file like this:  
```bash
complete -C '/home/crh/.local/lib/aws/bin/aws_completer' aws
```

Of course your aws_completer may be located in a different path, just update that.
