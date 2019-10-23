# WAF
### Install docker on bastion host

```
sudo apt install docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo docker run --rm -it -p 80:80 vulnerables/web-dvwa
```


### Find IP and open securitygroup
```
curl ifconfig.co
aws ec2 describe-security-groups --filters "Name=group-name,Values=bastion-sg" --query "SecurityGroups[].[GroupId]"
aws ec2 authorize-security-group-ingress --group-id GroupId --protocol tcp --port 80 --cidr YourIp
```

### Create alb and securitygroup
```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws ec2 create-security-group --group-name web-sg --description "web security group" --vpc-id VpcId
curl ifconfig.co
aws ec2 describe-security-groups --filters "Name=group-name,Values=web-sg" --query "SecurityGroups[].[GroupId]"
aws ec2 authorize-security-group-ingress --group-id GroupId --protocol tcp --port 80 --cidr YourIp
aws ec2 describe-security-groups --filters "Name=group-name,Values=bastion-sg" --query "SecurityGroups[].[GroupId]"
aws ec2 authorize-security-group-ingress --group-id GroupId --protocol tcp --port 80 --source-group amazon-elb-sg


aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.1.0/24" --query 'Subnets[].[SubnetId]'
aws ec2 describe-subnets --filters "Name=cidr-block,Values=10.0.2.0/24" --query 'Subnets[].[SubnetId]'
aws elbv2 create-load-balancer --name web-alb --subnets subnet-12345678 subnet-23456789 --security-groups GroupId
```

### Create targetgroup & register the targetgroup
```
aws ec2 describe-vpcs --filters "Name=cidr-block,Values=10.0.0.0/16" --query 'Vpcs[].[VpcId]'
aws elbv2 create-target-group --name dvwb-tg --protocol HTTP --port 80 --vpc-id vpc-12345678 --health-check-path /login.php
aws ec2 describe-instances --filters "Name=tag:Name,Values=bastion" --query "Reservations[].Instances[].[InstanceId]"
aws elbv2 register-targets --target-group-arn targetgroupArn --targets Id=instanceId
aws elbv2 create-listener --load-balancer-arn loadbalancerArn --protocol HTTP --port 80 --default-actions Type=forward,TargetGroupArn=targetgroupArn
```


### Test sql injection

```
http://albDNS/vulnerabilities/sqli/
%' or '0'='0
```

### Create needed matchsets/rules/acl's

```
aws waf-regional get-change-token
aws waf-regional create-sql-injection-match-set --name sqlmatch --change-token changeToken

aws waf-regional get-change-token
aws waf-regional update-sql-injection-match-set --sql-injection-match-set-id 4b0e0fcd-ca8d-40f7-9c43-92f21f4a60e0 --change-token changeToken --updates file://sqlinjection.json

aws waf-regional get-change-token
aws waf-regional create-rule --name sqlirule --metric-name sqlinjectionrule --change-token changeToken

aws waf-regional get-change-token
aws waf-regional list-sql-injection-match-sets
aws waf-regional get-change-token
aws waf-regional update-rule --rule-id ruleId --change-token changeToken --updates 'Action="INSERT",Predicate={Negated=false,Type="SqlInjectionMatch",DataId="sqlInjectionMatchSetsId"}'

aws waf-regional get-change-token
aws waf-regional create-web-acl --name web-sqli --metric-name websqli --default-action Type=ALLOW --change-token changeToken 

aws waf-regional get-change-token
aws waf-regional list-rules
aws waf-regional list-web-acls
aws waf-regional update-web-acl --web-acl-id webAclId --change-token changeToken --updates file://sqlinjectionwebacl.json

aws waf-regional list-web-acls
aws elbv2 describe-load-balancers --names web-alb --query "LoadBalancers[].[LoadBalancerArn]"

aws waf-regional associate-web-acl --web-acl-id WebACLId --resource-arn LoadBalancerArn
