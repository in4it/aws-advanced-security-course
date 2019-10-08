# IAM Database Authentication

### VPC db subnet group

```
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.4.0/24" --query 'Subnets[*].{SubnetId:SubnetId}'
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.5.0/24" --query 'Subnets[*].{SubnetId:SubnetId}'
aws rds create-db-subnet-group --db-subnet-group-name myDatabaseSubnetgroup --db-subnet-group-description "private subnetgroup for RDS instance" --subnet-ids SubnetId4 SubnetId5
```

### Create security group

```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 create-security-group --group-name training-rds-mysql --description "security group for training-rds-mysql" --vpc-id VpcId
aws ec2 describe-security-groups --filters "Name=group-name,Values=bastion-sg" --query "SecurityGroups[].[GroupId]"
aws ec2 authorize-security-group-ingress --group-id rdsInstanceSGId --protocol tcp --port 3306 --source-group BastionSGId
```

### Create MySQL RDS
The previous command outputs the GroupId (starts with sg-)

```
aws rds create-db-instance \
    --db-instance-identifier training-rds-mysql \
    --db-instance-class db.t2.micro \
    --engine MySQL \
    --allocated-storage 20 \
    --vpc-security-group-ids rdsInstanceSGId \
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

Get the accountID and DbiResourceId:
```
aws sts get-caller-identity --output text --query 'Account'
aws rds describe-db-instances --query "DBInstances[*].[DBInstanceIdentifier,DbiResourceId]"
```

```
aws iam put-role-policy --role-name bastion-role --policy-name rds-db-connect --policy-document file://policy-rds-connect.json
```

### Install mysql-client and postgresql-client on Bastion(EC2)

```
sudo apt update
sudo apt install mysql-client -y
```

### Create MySQL user from Bastion(EC2)

Connect to MySQL

```
DBHOST="training-rds-mysql.cd2dwqiadpid.eu-west-1.rds.amazonaws.com"
mysql --host=$DBHOST  \      
      --port=3306 \
      --user=masterawsuser \
      --password
```

```
CREATE DATABASE application;
CREATE USER traininguser IDENTIFIED WITH AWSAuthenticationPlugin AS 'RDS';
GRANT ALL PRIVILEGES ON application.* TO 'traininguser'@'%';
FLUSH PRIVILEGES;
```

### Connect to MySQL using the IAM role

```
curl -o rds-combined-ca-bundle.pem https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem
DBHOST="training-rds-mysql.cd2dwqiadpid.eu-west-1.rds.amazonaws.com"
TOKEN="$(aws rds generate-db-auth-token --hostname $DBHOST --port 3306 --username traininguser)"
mysql --host=$DBHOST \
      --port=3306 \
      --ssl-ca=rds-combined-ca-bundle.pem \
      --enable-cleartext-plugin \
      --user=traininguser \
      --password=$TOKEN
```

### Alter user masterawsuser in order to switch from password to IAM

```
ALTER USER masterawsuser IDENTIFIED WITH AWSAuthenticationPlugin AS 'RDS';
```

```
TOKEN="$(aws rds generate-db-auth-token --hostname $DBHOST --port 3306 --username masterawsuser)"
mysql --host=$DBHOST \
      --port=3306 \
      --ssl-ca=rds-combined-ca-bundle.pem \
      --enable-cleartext-plugin \
      --user=masterawsuser \
      --password=$TOKEN
```

# Cleanup

### Remove role

```
aws iam delete-role-policy --role-name bastion-role --policy-name rds-db-connect
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