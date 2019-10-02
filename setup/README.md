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

### Enable auto-assign public IPs

```
aws ec2 modify-subnet-attribute --subnet-id SubnetId1 --map-public-ip-on-launch --region eu-west-1
aws ec2 modify-subnet-attribute --subnet-id SubnetId2 --map-public-ip-on-launch --region eu-west-1
aws ec2 modify-subnet-attribute --subnet-id SubnetId3 --map-public-ip-on-launch --region eu-west-1
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

## Setup SSH keypair
### Create SSH keypair

```
aws ec2 create-key-pair --key-name MyTrainingKeyPair --query 'KeyMaterial' --region eu-west-1 --output text > MyTrainingKeyPair.pem
```

### Limit local access to the key

```
chmod 600 MyTrainingKeyPair.pem
```

### Check if key is created correctly

```
aws ec2 describe-key-pairs --key-name MyTrainingKeyPair --region eu-west-1
```

## Setup Security Groups

### Creating a Security Group

```
aws ec2 create-security-group --group-name bastion-sg --description "bastion security group" --region eu-west-1 --vpc-id VpcId
```

### Check Security Group
The previous command outputs the GroupId (starts with sg-)

```
aws ec2 describe-security-groups --region eu-west-1 --group-ids GroupId
```

### Check your ip

```
curl https://checkip.amazonaws.com
```

### Update security group to allow ingress SSH from your IP

```
aws ec2 authorize-security-group-ingress --group-id GroupId --protocol tcp --port 22 --region eu-west-1 --cidr YourIp
```

### Check Security Group updates
The previous command outputs the GroupId (starts with sg-)

```
aws ec2 describe-security-groups  --region eu-west-1 --group-ids GroupId
```

### Query for ami-id and subnet id

```
aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*" --query 'sort_by(Images,&CreationDate)[-1].ImageId' --region eu-west-1
```
```
aws ec2 describe-subnets --region eu-west-1 --filters "Name=cidr-block,Values=10.0.1.0/24" --query 'Subnets[*].{SubnetId:SubnetId}'
```

### Launch Bastion(EC2)
The previous commands output the amiId (starts with ami-) and the SubnetId (starts with subnet-)

```
aws ec2 run-instances --image-id amiId --count 1 --instance-type t2.micro --key-name MyTrainingKeyPair --security-group-ids GroupId --subnet-id GroupId
```


# Cleanup
```
aws ec2 delete-key-pair --key-name MyTrainingKeyPair
```