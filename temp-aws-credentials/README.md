# Temp iam credentials

### Install auth0-login
```
pip3 install auth0-login
```

### Change callback url in Auth0
```
http://localhost:12200/saml
```

### Create files
```
~/.saml-login
[DEFAULT]
idp_url = https://auth0-tenant.eu.auth0.com
client_id = your-newly-added-client-id
```

```
~/.aws-accounts
[DEFAULT]
aws-account-alias = aws-account-number
```

Test if all the configurations made are correct with the following command:
```
saml-login aws-assume-role --show
```

### Obtain AWS access keys

```
saml-login -c aws-role@aws-alias aws-assume-role
```

### Access the AWS console
```
aws-console --profile aws-role@aws-alias 
```