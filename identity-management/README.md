# Identity Management

## Setup
### Create SAML Provider

```
aws iam create-saml-provider --saml-metadata-document file://metadata.xml --name exampleSAMLProvider
```

### Setup IAM Role

```
aws iam create-role --role-name admin --assume-role-policy-document file://role-policy-saml.json --max-session-duration 28800 
```

### Setup admin policy for IAM role

```
aws iam attach-role-policy --role-name admin --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```
