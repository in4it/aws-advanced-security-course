# Network setup

# Launch EC2 instance (in private subnet)

```
aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*" --query 'sort_by(Images,&CreationDate)[-1].ImageId'
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[*].{VpcId:VpcId}'
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.4.0/24" --query 'Subnets[*].{SubnetId:SubnetId}'
aws ec2 create-security-group --group-name insternal-instance-sg --description "internal instance security group" --vpc-id VpcId
aws ec2 run-instances --image-id amiId --count 1 --instance-type t2.micro --key-name MyTrainingKeyPair --security-group-ids GroupId --subnet-id SubnetId4

```

# Setup NAT gateway (for egress traffic)
The first command gives you an eip allocation id (starts with eipalloc-)
```
aws ec2 allocate-address --domain vpc
aws ec2 create-nat-gateway --allocation-id eipAllocId --subnet-id SubnetId4
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
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Interface --service-name com.amazonaws.eu-west-1.kinesis --route-table-ids routeTableId
```


