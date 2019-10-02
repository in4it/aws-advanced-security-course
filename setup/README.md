# EC2 instance setup

## VPC 

### Create VPC

The following command outputs the VpcId (starts with vpc-)

```
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region eu-west-1
```

### VPC subnets

```
aws ec2 create-subnet --availability-zone eu-west-1a --cidr-block 10.0.1.0/24 --vpc-id VpcId --region eu-west-1
aws ec2 create-subnet --availability-zone eu-west-1b --cidr-block 10.0.2.0/24 --vpc-id VpcId --region eu-west-1
aws ec2 create-subnet --availability-zone eu-west-1c --cidr-block 10.0.3.0/24 --vpc-id VpcId --region eu-west-1
aws ec2 create-subnet --availability-zone eu-west-1a --cidr-block 10.0.4.0/24 --vpc-id VpcId --region eu-west-1
aws ec2 create-subnet --availability-zone eu-west-1b --cidr-block 10.0.5.0/24 --vpc-id VpcId --region eu-west-1
aws ec2 create-subnet --availability-zone eu-west-1c --cidr-block 10.0.6.0/24 --vpc-id VpcId --region eu-west-1
```

### Route table for public subnets
The first command outputs the RouteTableId (starts with rtb-)

```
aws ec2 create-route-table --vpc-id VpcId --region eu-west-1
aws ec2 associate-route-table --route-table-id RouteTableId --subnet-id SubnetId1 --region eu-west-1
aws ec2 associate-route-table --route-table-id RouteTableId --subnet-id SubnetId2 --region eu-west-1
aws ec2 associate-route-table --route-table-id RouteTableId --subnet-id SubnetId3 --region eu-west-1
```

### Internet Gateway

The first command outputs the InternetGatewayId (starts with igw-)

```
aws ec2 create-internet-gateway --region eu-west-1
aws ec2 attach-internet-gateway --internet-gateway-id InternetGatewayId --vpc-id VpcId --region eu-west-1
aws ec2 create-route --destination-cidr-block 0.0.0.0/0 --gateway-id InternetGatewayId --route-table-id RouteTableId --region eu-west-1
```

## EC2 instance


