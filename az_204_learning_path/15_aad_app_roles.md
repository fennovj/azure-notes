# Azure AD claims/roles

How to configure role claim in SAML token
Note that this stuff is used in Azure Web app too, except there, the enterprise app (service principal) is made automatically when enabling authentication. For apps hosted outside of webservice, or apps that don't use the web app auth, you have to implement the authentication yourself (or use a library that does it for you)

Very quick overview

- Step 1: create a enterprise app/service principal
- Step 2: give the required graph API permissions to the service principal
    - Note that each permission can be 'delegated' or 'application'. Delegated means it uses the permission of the user
    - A few more details: for stuff like reading email/username, it's obvious that a user is allowed to read their own email. For accessing a web app, there will generally be some middleware that allows anyone that is authenticated with the service principal. Then, you don't need additional delegated permissions to access the app itself. However, if the app needs to read e.g. email, you can use delegated permissions from the user, and use those to access the  email. The admin needs to give consent for this though, so you don't hijack people's data with some random phishing app.

## Tokens

Useful link: <https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-v2-protocols>

There are a bunch of different tokens:

- SAML tokens: These are the default tokens used in the SAML 2.0 protocol, which is used for the SSO in Azure. They are pretty long-ass tokens in XML. And I don't really understand the SAML protocol all that much...

- ID token: This is the authentication token used in OpenID connect flow. It shouldn't be used for authorization, but the name in it can be used to query the database, and change UX. They are JWT tokens, and with claims, can contain all kind of information.

- Access token: This is the authorization token, used to call APIs. clients don't have to decrypt this (and can't), they can just forward it to some protected API. The token contains some basic info like expiration date. The protected API should validate the token (using Azure AD middleware).

## Claims

Authentication kinda works like this: your app (or middleware) prompts client to login, Azure AD then sends a token to the app. The app can then use this token to authenticate instead of giving another login screen. This (SAML) token contains more info, known as *claims*. By default it just contains the `NameIdentifier`, or UPN. It can optionally contain stuff like email, first name, last name.

You can configure claims on the app registration, under 'Token configuration'. This is all basic profile information, such as email, nickname, password expiration date, ip address. (Note, for our app, even though we didn't configure any optional claims, it still sent the upn, given_name and family_name).

This configuration can be done by clicking some checkboxes, or programatically in JSON. You can use this JSON with some rest endpoint probably, it ends up in the manifest for the app registration. It looks like:

```json
"optionalClaims": {
    "idToken": [
        {
            "name": "auth_time",
            "essential": false
        }
    ],
    "accessToken": [
        {
            "name": "ipaddr",
            "essential": false
        }
    ],
    "saml2Token": [
        {
            "name": "upn",
            "essential": false,
            "additionalProperties": [
                "include_externally_authenticated_upn"
            ]
        },
        {
            "name": "extension_ab603c56068041afb2f6832e2a17e237_skypeId",
            "source": "user",
            "essential": false
        }
    ]
}
```

Here is a list of optiopnal claims: <https://docs.microsoft.com/en-us/azure/active-directory/develop/active-directory-optional-claims>

Note: the additional properties is mainly for migration of on-premise app, you can configure if you want UPN as email@extdomain.com#EXT#@bigfellows.onmicrosoft.com, or (by default) just as email@extdomain.com

## Permissions and consent

These are what to use to do stuff like microsoft graph, reading outlook, getting keyvault secrets on behalf of user. I am not sure if you can do storage accounts, but that's not really the intent anyway, it's mainly for data specific to that user, rather than general application data that is for multiple users.

The service principal *itself* can call the microsoft graph api for some info. It uses OAuth2.0.

- Delegated permissions: user (or admin) consents, then you can access the relevant graph api. Your app can never have more privileges than the user themselves.
- Application: These can be run without any user present, and always require admin consent.

### Using permissions

You need to get an access token, using the microsoft identity platform endpoint, you send a post request there with your JWT token, and it sends back a token to use to acess the graph api.

## Implicit flow

Implicit flow is a specific kind of OAuth2.0 flow that allows you to authenticate in javascript in the browser only, mainly for SPA (react, angular, etc). It's kinda less secure, but sometimes neccecary. This has to be turned on in the app manifest.
