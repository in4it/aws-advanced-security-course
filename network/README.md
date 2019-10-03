# Network setup

# Launch EC2 instance (in private subnet)

## Add IAM Role
```
aws iam create-role --role-name internal-instance --assume-role-policy-document file://ec2-role-policy.json  
aws iam attach-role-policy --role-name internal-instance --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam attach-role-policy --role-name internal-instance --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
aws iam attach-role-policy --role-name internal-instance --policy-arn arn:aws:iam::aws:policy/AmazonKinesisFullAccess
aws iam create-instance-profile --instance-profile-name internal-instance
aws iam add-role-to-instance-profile --instance-profile-name internal-instance --role-name internal-instance
```

## Launch instance
```
aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*" --query 'sort_by(Images,&CreationDate)[-1].ImageId'
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.4.0/24" --query 'Subnets[].[SubnetId]'
aws ec2 create-security-group --group-name internal-instance-sg --description "internal instance security group" --vpc-id VpcId
aws ec2 describe-security-groups --filters "Name=group-name,Values=bastion-sg" --query "SecurityGroups[].[GroupId]"
aws ec2 authorize-security-group-ingress --group-id internalInstanceSGId --protocol tcp --port 22 --source-group BastionSGId
aws ec2 run-instances --iam-instance-profile "Name=internal-instance" --image-id amiId --count 1 --instance-type t2.micro --key-name MyTrainingKeyPair --security-group-ids GroupId --subnet-id SubnetId4 --tag-specifications "ResourceType=instance,Tags=[{Key=Name,Value=internal-instance}]"
```

# Setup NAT gateway (for egress traffic)
The first command gives you an eip allocation id (starts with eipalloc-)
```
aws ec2 allocate-address --domain vpc
aws ec2 create-nat-gateway --allocation-id eipAllocId --subnet-id SubnetId1
aws ec2 describe-route-tables --filters "Name=route.destination-cidr-block,Values=10.0.0.0/16" --filter "Name=association.main,Values=true"
aws ec2 create-route --destination-cidr-block 0.0.0.0/0 --route-table-id RouteTableId --nat-gateway-id VpcNatId

```

## Host names lookup
```
aws s3 ls --debug
host s3.eu-west-1.amazonaws.com
aws kinesis list-streams --debug
host kinesis.eu-west-1.amazonaws.com
```

## VPC endpoints

### Dynamodb endpoint

```
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Gateway --service-name com.amazonaws.eu-west-1.dynamodb --route-table-ids routeTableId
```

### S3 endpoint

```
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Gateway --service-name com.amazonaws.eu-west-1.s3 --route-table-ids routeTableId
```

### Kinesis endpoint

```
aws ec2 create-security-group --group-name kinesis-endpoint-sg --description "kinesis endpoint security group" --vpc-id VpcId
aws ec2 authorize-security-group-ingress --group-id kinesisEndointSGId --protocol tcp --port 443 --cidr 10.0.0.0/16
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.4.0/24" --query 'Subnets[*].{SubnetId:SubnetId}'
aws ec2 create-vpc-endpoint --security-group-ids SecurityGroupId --vpc-id VpcId --vpc-endpoint-type Interface --service-name com.amazonaws.eu-west-1.kinesis-streams --subnet-ids Subnet4
```


# Cleanup
## Endpoints
```
aws ec2 describe-vpc-endpoints --query "VpcEndpoints[].[VpcEndpointId]"
aws ec2 delete-vpc-endpoints --vpc-endpoint-ids DynamoDBVPCEndpointID S3EndpointID EndpointID
```

## NAT gw
```
aws ec2 describe-nat-gateways --query "NatGateways[*].{NatGatewayId:NatGatewayId}"
aws ec2 delete-nat-gateway --nat-gateway-id NatGwId
```
## EC2 instance
```
aws ec2 describe-instances --filters "Name=tag:Name,Values=internal-instance" --query "Reservations[].Instances[].[InstanceId]"
aws ec2 terminate-instances --instance-ids InstanceIds
```

## Roles
```
aws iam remove-role-from-instance-profile --instance-profile-name internal-instance --role-name internal-instance
aws iam delete-instance-profile --instance-profile-name internal-instance
aws iam detach-role-policy --role-name internal-instance --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
aws iam detach-role-policy --role-name internal-instance --policy-arn arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess
aws iam detach-role-policy --role-name internal-instance --policy-arn arn:aws:iam::aws:policy/AmazonKinesisFullAccess
aws iam delete-role --role-name internal-instance
```

## Security groups
```
aws ec2 describe-security-groups --filters "Name=group-name,Values=internal-instance-sg,kinesis-endpoint-sg" --query "SecurityGroups[].[GroupId]"
aws ec2 delete-security-group --group-id InternalInstanceSecurityGroupId
aws ec2 delete-security-group --group-id KinesisSecurityGroupId
```

## Routes
```
aws ec2 describe-route-tables --filters "Name=route.state,Values=blackhole"
aws ec2 delete-route --route-table-id routeTableId --destination-cidr-block 0.0.0.0/0
```
