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

### Create Auth0 action

```
exports.onExecutePostLogin = async (event, api) => {
  if(context.clientID === "8SERXtt0QXhMXz2wFMtY8BQevQgpKxQH") {
    api.samlResponse.setAttribute('https://aws.amazon.com/SAML/Attributes/Role', 'arn:aws:iam::AccountId:role/admin-auth0,arn:aws:iam::AccountId:saml-provider/auth0samlprovider')
    api.samlResponse.setAttribute('https://aws.amazon.com/SAML/Attributes/RoleSessionName', event.user.email)
  }
}
```
