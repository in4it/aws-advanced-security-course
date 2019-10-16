# S3 vpc-locked IAM

## VPC locked
### S3 endpoint

```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=VpcId"
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Gateway --service-name com.amazonaws.eu-west-1.s3 --route-table-ids routeTableId
```

### Create bucket

```
aws s3api create-bucket --bucket s3trainingbucketjornj --region eu-west-1 --create-bucket-configuration LocationConstraint=eu-west-1
```

### Test file upload locally

```
touch testfilelocallyopen.txt
aws s3 cp testfilelocallyopen.txt s3://s3trainingbucketjornj/testfilelocallyopen.txt
aws s3 ls s3://s3trainingbucketjornj/
```

### Bucket policy (Lock on VPC)

```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws s3api put-bucket-policy --bucket s3trainingbucketjornj --policy file://policy-s3-lock-vpc.json
```

### Allow bastion role to access s3

```
aws iam put-role-policy --role-name bastion-role --policy-name ec2-s3-access --policy-document file://policy-s3-lock-ec2.json
```

### Test file upload locally

```
touch testfilelocally.txt
aws s3 cp testfilelocally.txt s3://s3trainingbucketjornj/testfilelocally.txt
aws s3 ls s3://s3trainingbucketjornj/
```

### Test file upload from Bastion(EC2)

```
touch testfilebastionvpc.txt
aws s3 cp testfilebastionvpc.txt s3://s3trainingbucketjornj/testfilebastionvpc.txt
aws s3 ls s3://s3trainingbucketjornj/
```

### Test file upload from internal-instance(EC2)

```
touch testfileinternalvpc.txt
aws s3 cp testfileinternalvpc.txt s3://s3trainingbucketjornj/testfileinternalvpc.txt
aws s3 ls s3://s3trainingbucketjornj/
```

## VPC endpoint locked

### Attach iam role to bastion (locally)

```
aws iam attach-role-policy --role-name bastion-role --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

### Cleanup policy (Bastion)

```
aws s3api delete-bucket-policy --bucket s3trainingbucketjornj
```

### Detach iam role to bastion (locally)

```
aws iam detach-role-policy --role-name bastion-role--policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess
```

### S3 endpoint

```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=VpcId"
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Gateway --service-name com.amazonaws.eu-west-1.s3 --route-table-ids routeTableId
```

### Bucket policy (Lock on VPCE)

```
aws ec2 describe-vpc-endpoints --query "VpcEndpoints[].[VpcEndpointId]"
aws s3api put-bucket-policy --bucket s3trainingbucketjornj --policy file://policy-s3-lock-vpce.json
```

### Test file upload from Bastion(EC2)

```
touch testfilebastion.txt
aws s3 cp testfilebastion.txt s3://s3trainingbucketjornj/testfilebastion.txt
aws s3 ls s3://s3trainingbucketjornj/
```

### Test file upload from internal-instance(EC2)

```
touch testfileinternalvpcendpoint.txt
aws s3 cp testfileinternalvpcendpoint.txt s3://s3trainingbucketjornj/testfileinternalvpcendpoint.txt
aws s3 ls s3://s3trainingbucketjornj/
```

# Cleanup

### Remove bucket (from internal instance)

```
aws s3api delete-bucket --bucket s3trainingbucketjornj
```

### Remove endpoints (locally)

```
aws ec2 describe-vpc-endpoints --query "VpcEndpoints[].[VpcEndpointId]"
aws ec2 delete-vpc-endpoints --vpc-endpoint-ids S3EndpointID
```

## EC2 instance (locally)

```
aws ec2 describe-instances --filters "Name=tag:Name,Values=internal-instance" --query "Reservations[].Instances[].[InstanceId]"
aws ec2 terminate-instances --instance-ids InstanceIds
```