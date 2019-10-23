# SSO auth0

### Setup auth0 app

```
https://signin.aws.amazon.com/saml
```
```
{
  "audience": "https://signin.aws.amazon.com/saml",
  "mappings": {
    "email": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress",
    "name": "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name"
  },
  "createUpnClaim": false,
  "passthroughClaimsWithNoMapping": false,
  "mapUnknownClaimsAsIs": false,
  "mapIdentities": false,
  "nameIdentifierFormat": "urn:oasis:names:tc:SAML:2.0:nameid-format:persistent",
  "nameIdentifierProbes": [
    "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress"
  ]
}
```

### Create saml provider

```
aws iam create-saml-provider --saml-metadata-document file://downloadedMetadataFile.xml --name auth0samlprovider
```


### Create iam role

```
aws sts get-caller-identity --output text --query 'Account'
aws iam create-role --role-name admin-auth0 --assume-role-policy-document file://assume-role-policy.json
aws iam attach-role-policy --role-name internal-instance --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
```

### Create Auth0 rule

```
function (user, context, callback) {
  if(context.clientID === "8SERXtt0QXhMXz2wFMtY8BQevQgpKxQH") {
    user.awsRole = 'arn:aws:iam::AccountId:role/admin-auth0,arn:aws:iam::AccountId:saml-provider/auth0SamlProvider';
    user.awsRoleSession = user.email;

    context.samlConfiguration.mappings = {
      'https://aws.amazon.com/SAML/Attributes/Role': 'awsRole',
      'https://aws.amazon.com/SAML/Attributes/RoleSessionName': 'awsRoleSession'
    };
  }
  callback(null, user, context);

}
```