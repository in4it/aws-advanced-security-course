# Egress traffic

## Download ip-ranges.json
```
wget https://ip-ranges.amazonaws.com/ip-ranges.json
```

For more info on how to query the file, see https://docs.aws.amazon.com/general/latest/gr/aws-ip-ranges.html

For more information about the limits, see https://docs.aws.amazon.com/vpc/latest/userguide/amazon-vpc-limits.html

## Delete egress rule

```
aws ec2 describe-security-groups --filters "Name=group-name,Values=internal-instance-sg" --query "SecurityGroups[].[GroupId]"
aws ec2 revoke-security-group-egress --group-id InternalInstanceSGId --cidr 0.0.0.0/0 --protocol -1
```

## Add new rules

```
aws ec2 authorize-security-group-egress --group-id internalInstanceSGId --protocol -1 --cidr 10.0.0.0/16
aws ec2 authorize-security-group-egress --group-id internalInstanceSGId --protocol -1 --cidr 8.8.8.8/32
```

## Add rules for dynamodb/s3

```
aws ec2 describe-prefix-lists
aws ec2 authorize-security-group-egress --group-id internalInstanceSGId --ip-permissions FromPort=443,ToPort=443,IpProtocol=tcp,PrefixListIds=[{PrefixListId=plForDynamodb}]
aws ec2 authorize-security-group-egress --group-id internalInstanceSGId --ip-permissions FromPort=443,ToPort=443,IpProtocol=tcp,PrefixListIds=[{PrefixListId=plForS3}]
```

## Setup Forward Proxy

* Start CloudFormation with template https://raw.githubusercontent.com/in4it/forward-proxy/master/forward-proxy-cloudformation.template

## Fix .local resolving
```
rm /etc/resolv.conf
ln -sf /run/systemd/resolve/resolv.conf /etc/resolv.conf
```

