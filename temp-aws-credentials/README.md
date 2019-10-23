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
saml-login aws-assume-role --show
```

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

### Obtain AWS access keys

```
saml-login -c OAuthAdministrator@aws-alias aws-assume-role
```

### Access the AWS console
```
aws-console --profile OAuthAdministrator@aws-alias 
```