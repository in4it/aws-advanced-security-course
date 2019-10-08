# S3 vpc-locked IAM

### S3 endpoint

```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 describe-route-tables --filters "Name=vpc-id,Values=VpcId"
aws ec2 create-vpc-endpoint --vpc-id VpcId --vpc-endpoint-type Gateway --service-name com.amazonaws.eu-west-1.s3 --route-table-ids routeTableId
```

### Create bucket

```
aws s3api create-bucket --bucket s3trainingbucketjorn
```

### Test file upload

```
touch testfile.txt
aws s3 cp testfile.txt s3://s3trainingbucketjorn/testfile.txt
```

### Bucket policy

```
aws s3api put-bucket-policy --bucket s3trainingbucketjorn --policy file://policy-s3-lock.json
```


### Test file upload locally

```
touch testfile.txt
aws s3 cp testfile.txt s3://s3trainingbucketjorn/testfile.txt
```

### Test file upload from Bastion(EC2)

```
touch testfile.txt
aws s3 cp testfile.txt s3://s3trainingbucketjorn/testfile.txt
```

# Cleanup

### Remove bucket
```
aws s3api delete-bucket s3trainingbucketjorn
```