# Setup

## AWS Account setup

* AWS CLI installation (Windows): https://docs.aws.amazon.com/cli/latest/userguide/install-windows.html
* AWS CLI installation (MacOS: https://docs.aws.amazon.com/cli/latest/userguide/install-macos.html

## EC2 instance setup

## VPC 

### Create VPC

The first command outputs the VpcId (starts with vpc-)

```
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 modify-vpc-attribute --enable-dns-hostnames --vpc-id VpcId
```

### VPC subnets

```
aws ec2 create-subnet --availability-zone eu-west-1a --cidr-block 10.0.1.0/24 --vpc-id VpcId
aws ec2 create-subnet --availability-zone eu-west-1b --cidr-block 10.0.2.0/24 --vpc-id VpcId
aws ec2 create-subnet --availability-zone eu-west-1c --cidr-block 10.0.3.0/24 --vpc-id VpcId
aws ec2 create-subnet --availability-zone eu-west-1a --cidr-block 10.0.4.0/24 --vpc-id VpcId
aws ec2 create-subnet --availability-zone eu-west-1b --cidr-block 10.0.5.0/24 --vpc-id VpcId
aws ec2 create-subnet --availability-zone eu-west-1c --cidr-block 10.0.6.0/24 --vpc-id VpcId
```

### Enable auto-assign public IPs

```
aws ec2 modify-subnet-attribute --subnet-id SubnetId1 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id SubnetId2 --map-public-ip-on-launch
aws ec2 modify-subnet-attribute --subnet-id SubnetId3 --map-public-ip-on-launch
```

### Route table for public subnets
The first command outputs the RouteTableId (starts with rtb-)

```
aws ec2 create-route-table --vpc-id VpcId
aws ec2 associate-route-table --route-table-id RouteTableId --subnet-id SubnetId1
aws ec2 associate-route-table --route-table-id RouteTableId --subnet-id SubnetId2
aws ec2 associate-route-table --route-table-id RouteTableId --subnet-id SubnetId3
```

### Internet Gateway

The first command outputs the InternetGatewayId (starts with igw-)

```
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id InternetGatewayId --vpc-id VpcId
aws ec2 create-route --destination-cidr-block 0.0.0.0/0 --gateway-id InternetGatewayId --route-table-id RouteTableId
```

## EC2 instance

## Setup SSH keypair
### Create SSH keypair

```
aws ec2 create-key-pair --key-name MyTrainingKeyPair --query 'KeyMaterial' --output text > MyTrainingKeyPair.pem
```

### Limit local access to the key

```
chmod 600 MyTrainingKeyPair.pem
```

### Check if key is created correctly

```
aws ec2 describe-key-pairs --key-name MyTrainingKeyPair
```

## Setup Security Groups

### Creating a Security Group

```
aws ec2 create-security-group --group-name bastion-sg --description "bastion security group" --vpc-id VpcId
```

### Check Security Group
The previous command outputs the GroupId (starts with sg-)

```
aws ec2 describe-security-groups --group-ids GroupId
```

### Check your ip

```
curl https://checkip.amazonaws.com
```

### Update security group to allow ingress SSH from your IP

```
aws ec2 authorize-security-group-ingress --group-id GroupId --protocol tcp --port 22 --cidr YourIp
```

### Check Security Group updates
The previous command outputs the GroupId (starts with sg-)

```
aws ec2 describe-security-groups  --group-ids GroupId
```

## Setup IAM

### Create role for instance profile for Bastion(EC2)

```
aws iam create-role --role-name bastion-role --assume-role-policy-document file://role-policy.json
```

### Create instance profile for Bastion(EC2)

```
aws iam create-instance-profile --instance-profile-name bastion-role
aws iam add-role-to-instance-profile --instance-profile-name bastion-role --role-name bastion-role
```

## Setup EC2 instance

### Query for ami-id and subnet id

```
aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*" --query 'sort_by(Images,&CreationDate)[-1].ImageId'
```
```
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.1.0/24" --query 'Subnets[*].{SubnetId:SubnetId}'
```

### Launch Bastion(EC2)
The previous commands outputs the amiId (starts with ami-) and the SubnetId (starts with subnet-)

```
aws ec2 run-instances --iam-instance-profile Name=bastion-role --image-id amiId --count 1 --instance-type t2.micro --key-name MyTrainingKeyPair --security-group-ids GroupId --subnet-id GroupId
```

### Tag Bastion(EC2)

The previous command outputs the InstanceId (starts with i-)

```
aws ec2 create-tags --resources instanceID --tags 'Key="Name",Value=bastion'
```

### Test ssh

```
ssh -i MyTrainingKeyPair.pem ubuntu@publicIpBastion
```

### Install and configure aws cli

```
sudo apt update
sudo apt install python3-pip
pip install --upgrade --user awscli
```

# Cleanup to remove all resources after the labs
```
aws ec2 delete-key-pair --key-name MyTrainingKeyPair
aws iam remove-role-from-instance-profile --instance-profile-name bastion --role-name rds-db-connect
```
