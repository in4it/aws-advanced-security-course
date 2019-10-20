# DynamoDB vpc-locked IAM

## VPC locked
### DynamoDB endpoint

```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=VpcId"
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Gateway --service-name com.amazonaws.eu-west-1.dynamodb --route-table-ids routeTableId
```

### Create table

```
aws dynamodb create-table --table-name CarCollection --attribute-definitions AttributeName=Brand,AttributeType=S AttributeName=Type,AttributeType=S --key-schema AttributeName=Brand,KeyType=HASH AttributeName=Type,KeyType=RANGE --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
```

### Put record locally

```
aws dynamodb put-item --table-name CarCollection --item file://caraudi.json --return-consumed-capacity TOTAL
```

### Allow bastion role to access DynamoDB

```
aws sts get-caller-identity --output text --query 'Account'
aws iam put-role-policy --role-name admin --policy-name deny-dynamodb-access-outside-vpc --policy-document file://policy-dynamodb-lock-vpc.json
aws iam put-role-policy --role-name bastion-role --policy-name ec2-dynamodb-access-vpc --policy-document file://policy-dynamodb-ec2.json
aws iam put-role-policy --role-name bastion-role --policy-name deny-dynamodb-access-outside-vpc --policy-document file://policy-dynamodb-lock-vpc.json
```

### Put record locally and from bastion

```
aws dynamodb put-item --table-name CarCollection --item file://carbmw.json --return-consumed-capacity TOTAL
```

## VPCe locked

### Cleanup IAM roles

```
aws iam delete-role-policy --role-name admin --policy-name deny-dynamodb-access-outside-vpc
aws iam delete-role-policy --role-name bastion-role --policy-name deny-dynamodb-access-outside-vpc
```

### DynamoDB endpoint in internal subnet

```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.4.0/24" --query 'Subnets[*].[SubnetId]'
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=VpcId"
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Gateway --service-name com.amazonaws.eu-west-1.dynamodb --route-table-ids routeTableId
```


### Allow internal-instance role to access DynamoDB

```
aws sts get-caller-identity --output text --query 'Account'
aws iam put-role-policy --role-name bastion-role --policy-name deny-dynamodb-access-vpce --policy-document file://policy-dynamodb-lock-vpce.json
aws iam put-role-policy --role-name internal-instance --policy-name ec2-dynamodb-access --policy-document file://policy-dynamodb-ec2.json
aws iam put-role-policy --role-name internal-instance --policy-name deny-dynamodb-access-vpce --policy-document file://policy-dynamodb-lock-vpce.json
```

### Put record from bastion and internal instance

```
aws dynamodb put-item --table-name CarCollection --item file://carbmw.json --return-consumed-capacity TOTAL
```

# Cleanup
## Endpoints
```
aws ec2 describe-vpc-endpoints --query "VpcEndpoints[].[VpcEndpointId]"
aws ec2 delete-vpc-endpoints --vpc-endpoint-ids DynamoDBVPCEndpointID 
```

## NAT gw
```
aws ec2 describe-nat-gateways --query "NatGateways[*].{NatGatewayId:NatGatewayId}"
aws ec2 delete-nat-gateway --nat-gateway-id NatGwId
```

## EC2 instance
```
aws ec2 describe-instances --filters "Name=tag:Name,Values=internal-instance" --query "Reservations[].Instances[].[InstanceId]"
aws ec2 describe-instances --filters "Name=tag:Name,Values=bastion" --query "Reservations[].Instances[].[InstanceId]"
aws ec2 terminate-instances --instance-ids InstanceIds
```

## Roles
```
aws iam remove-role-from-instance-profile --instance-profile-name internal-instance --role-name internal-instance
aws iam delete-instance-profile --instance-profile-name internal-instance
aws iam delete-role --role-name internal-instance
```