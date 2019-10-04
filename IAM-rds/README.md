# IAM Database Authentication

### VPC db subnets

```
aws rds create-db-subnet-group --db-subnet-group-name myDatabaseSubnetgroup --db-subnet-group-description "private subnetgroup for RDS instance" --subnet-ids SubnetId4 SubnetId5
```

### Create Security group

```
aws ec2 create-security-group --group-name training-rds-mysql --description "security group for training-rds-mysql" --vpc-id VpcId
```

### Create MySQL RDS
The previous command outputs the GroupId (starts with sg-)

```
aws rds create-db-instance \
    --db-instance-identifier training-rds-mysql \
    --db-instance-class db.t2.micro \
    --engine MySQL \
    --allocated-storage 20 \
    --vpc-security-group-ids GroupId \
    --db-subnet-group-name myDatabaseSubnetgroup \
    --master-username masterawsuser \
    --master-user-password USESECUREPASSWORD \
    --enable-iam-database-authentication 
```

### Modify MySQL RDS

```
aws rds modify-db-instance \
    --db-instance-identifier training-rds-mysql \
    --apply-immediately \
    --enable-iam-database-authentication
```

### Create and attach policy to the IAM role to allow rds-db:connect from bastion

The previous command outputs the DbiResourceId (starts with db-)

Get the accountID:
```
aws sts get-caller-identity --output text --query 'Account'
```

```
aws iam put-role-policy --role-name bastion-role --policy-name rds-db-connect --policy-document file://policy-rds-connect.json
```

### Create securtiy group to allow MySQL traffic and attach to RDS

```
aws ec2 create-security-group --group-name bastion-mysql-sg --description "bastion mysql security group" --vpc-id VpcId


```

### Install mysql-client and postgresql-client on Bastion(EC2)

```
sudo apt update
sudo apt install mysql-client postgresql-client -y
```

### Create MySQL user from Bastion(EC2)
```
CREATE USER dba_admin IDENTIFIED WITH AWSAuthenticationPlugin AS 'RDS';
```

# Cleanup

### Remove role

```
aws iam delete-role --role-name rds-db-connect
```

### Remove RDS with snapshot

```
aws rds delete-db-instance \
    --db-instance-identifier training-rds-mysql \
    --final-db-snapshot-identifier training-rds-mysql-final-snap
```

### Remove RDS without snapshot

```
aws rds delete-db-instance \
    --db-instance-identifier training-rds-mysql \
    --skip-final-snapshot
```
