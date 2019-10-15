# AWS SSM SSH setup

## IAM perms

```
aws iam attach-role-policy --role-name bastion-role --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```


## Install session manager
* See https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html#install-plugin-macos

## Connect using CLI
```
aws ssm start-session --target instance-id
````
