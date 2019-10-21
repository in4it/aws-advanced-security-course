# IAM boundaries

### Setup IAM Company Boundary Role 

```
aws iam create-policy --policy-name Company-Boundary --policy-document file://CompanyBoundary.json
aws iam create-policy --policy-name Delegated-User-Boundary --policy-document  file://DelegatedUserBoundary.json
```

### Setup IAM Delegate dUser Boundary Role 

```
aws iam list-policies --query 'Policies[?PolicyName==`Delegated-User-Boundary`].Arn'
aws iam put-role-permissions-boundary --role-name admin --permissions-boundary arn:aws:iam::AccountId:policy/Delegated-User-Boundary
```

### Test policy

```
aws iam create-user --user-name testuser
```

```
aws iam list-policies --query 'Policies[?PolicyName==`Delegated-User-Boundary`].Arn'
aws iam create-user --user-name testuser --permissions-boundary arn:aws:iam::AccountId:policy/Delegated-User-Boundary
```