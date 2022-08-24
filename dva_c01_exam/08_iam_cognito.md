# IAM and Cognito

## Cognito User pool vs Identity Pool

A user pool is a list of users. Every user has a 'user profile', but users can log in through Socials or Custom SAML. Cognito issues JWT tokens to users that they can use to authorize to API's (API Gateway), or retrieve AWS Credentials (with identity pool, below). User pools do not allow guest access (since it has no concept of 'guests').

An identity pool is something that provides AWS Credentials. It can do either social logins, cognito users or guests. You can define both an 'authenticated' and 'unauthenticated' role to let users/guests assume.

I still don't fully get it, but the basic process is:

1) Login to User Pool (socials, SALM, etc)
1.5) Get JWT token from User Pool
If your app just uses JWT Token and doesn't need guest access, you can stop here?

2) Obtain AWS Credentials from Identity pool
2.5) If you're a guest, skip step 1. Or sometimes you can also authenticate with identity pool directly?
3) Access AWS Services/API Gateway/etc.

## IAM Requesting temporary credentials

<https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_request.html>

You can make different STS calls to get temporary AWS credentials (access/secret key)

- AssumeRole - If you already have valid credentials, you just need new credentials for another role. Mostly for use in safe environments, e.g. on EC2, or on developer machines on safe networks.
- AssumeRoleWithSAML - Given SAML authentication, get some short lived credentials. Use this if you want to give credentials to people through SSO, e.g. if your company use Active Directory or LDAP.
- AssumeRoleWithWebIdentity - Given web identity like Google/Facebook/etc, give short lived credentials. Generally this is not preferred, you should use Cognito identity pool to give credentials to users.
- GetFederationToken - Federation token is like a role but permissions are managed by the federation token. This means you do access management in your own federation system, and the federation decides which resources you get access to. (the role you use will just be some administrator that can do everything, but the federation decides what to give out)
- GetSessionToken - Similar to AssumeRole but downscoped with shorter credentials/shorter duration. This is used when you e.g. want to hand out a short-lived token in an insecure environment. E.g. what we use in Quby - a safe central server calls 'GetSessionToken', and hands out the session tokens to developers to use in unsafe environments such as local unmanaged machines.

## Cognito Sync

Service that allows you to sync user data across devices.

It uses a local cache (meaning any local change is immediately locally available, even when device is offline). When `synchronize` is called, it pulls changes from cloud, and pushes changes to cloud.

There is also `AppSync` that does the same thing, but also allows multiple users to synchronize shared data.