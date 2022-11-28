# Relying Party Server

## Overview

The relying-party-server is the server-side implementation to support WebAuthn solutions with IBM Security Verify.  In addition, the relying-party-server exposes endpoints that proxy requests to IBM Security Verify OAuth token services.

### Getting started

Create a free trial tenant here: https://www.ibm.com/account/reg/us-en/signup?formid=urx-30041. 

You'll need to have an IBMid but this can be done at the same time.  

This link explains setting up your tenant: https://docs.verify.ibm.com/verify/docs/signing-up-for-a-free-trial


```
git clone https://github.com/craigaps/relying-party-server.git
```

## Contents

### Environment variables
The relying-party-server requires a number of environment variables to launch.


#### `APPLE_APP_SITE_ASSOC`

This is a string to represent the JSON for Apple to establish a secure association between domains and your app.  The following JSON code represent the contents of a simple association:
```
{
    "webcredentials":{
        "apps":[
            "ABCDE12345.com.example.app"
        ]
    }
}
```

> The JSON content should be minified when assigning to the environment variable.


See [Supporting associated domains](https://developer.apple.com/documentation/xcode/supporting-associated-domains) for more information.

#### `FIDO2_RELYING_PARTY_ID`

This is the unique identifier (UUID) that is created when the FIDO2 service is created in IBM Security Verify.  For example:
```
634cd513-dc6a-5e28-06fg-40c3dc81a79e
```

#### `API_CLIENT_ID` and `API_CLIENT_SECRET`

This is the unique client identifier and confidential client secret that the relying party server uses internally to establlished an authenticated session with the FIDO2 and factors endpoints.

See [FIDO2](https://docs.verify.ibm.com/verify/docs/fido2-login) for more information.

#### `AUTH_CLIENT_ID` and `AUTH_CLIENT_SECRET`

This is the unique client identifier and confidential client secret that the relying party server uses internally to establlished an authenticated session with the OIDC token endpoints.

See [Client Credentials](https://docs.verify.ibm.com/verify/docs/get-an-access-token) for more information.

#### `BASE_URL`

The base URL is the fully-qualified hostname of your tenant.  For example:
```
https://example.verify.ibm.com
``` 

### Endpoints

#### `GET /.well-known/apple-app-site-assoication`

Returns the JSON content representing the `APPLE_APP_SITE_ASSOC` environment variable.


#### `POST /v1/authenticate`

Used for existing accounts with a password resulting in an ROPC request to token endpoint.  Below is a sample request payload:

```
{
    "email": "anne_johnson@icloud.com",
    "password": "a1b2c3d4"
}
```

If successful, the response format is as follows:
```
{
    "id_token": "eyJ0eXA.2NDUxMjV9.5Od-8LjVM",
    "token_type": "Bearer",
    "access_token": "6ImNsb3VkSWRlbnRpdHlSZW",
    "expires_in": 604800
}
```

#### `POST /v1/signup`

Allows a new account to be created where ownership of an email is validated.  Below is a sample request payload:

```
{
    "name": "Anne Johnson", 
    "email": "anne_johnson@icloud.com"
}
```

If successful, the response format is as follows:
```
{
    "expiry": "2022-11-28T12:26:34Z",
    "correlation": "1719",
    "transactionId": "95f36a22-558a-438b-bdac-1490f279bb0d"
}
```

#### `POST /v1/validate`

Validate the one-time password generated by the `signup`.  Below is a sample request payload:

```
{
    "transactionId": "95f36a22-558a-438b-bdac-1490f279bb0d",
    "otp": "12345"
}
```

If successful, the response format is as follows:
```
{
    "id_token": "eyJ0eXA.2NDUxMjV9.5Od-8LjVM",
    "token_type": "Bearer",
    "access_token": "6ImNsb3VkSWRlbnRpdHlSZW",
    "expires_in": 604800
}
```

#### `POST /v1/register`

Registers a new public-key credential for a user.  Below is a sample request payload:

```
{
    "nickname": "John's iPhone",
    "clientDataJSON": "eyUyBg8Li8GH...",
    "attestationObject": "o2M884Yt0a3B7...",
    "credentialId": "VGhpcyBpcyBh..."
}
```

If successful, the response status a `201 Created`.

> The `access_token` must be presented in the request authorization header.  For example:
>```
>Authorization: Bearer NLL8EtOJFdbPiwPwZ
>```
> A `401 Unauthorized` will result otherwise.

#### `POST /v1/challenge`

Generates a new challenge for perform WebAuthn  registrations and verifications.  

**Verification (assertion)**

Below is a sample request payload for a assertion (signin):

```
{
    "type": "assertion"
}
```

If successful, the response format is as follows:
```
{
    "challenge": "4l96-NXQ8AZHUwhSlHHqesjW4rCXV6O566EF74qbtOI"
}
```

**Registration (attestation)**

Below is a sample request payload for a attestation:

```
{
    "displayName": "Jane's iPhone",
    "type": "assertion"
}
```

If successful, the response format is as follows:
```
{
    "challenge": "nKHUEOcZYgnhYOGLO50knadtWMv8Gka9rAsp8vnbLWo"
}
```

> The `access_token` must be presented in the request authorization header.  For example:
>```
>Authorization: Bearer NLL8EtOJFdbPiwPwZ
>```
> A `401 Unauthorized` will result otherwise.

#### `POST /v1/signin`

Validates a public-key credential for a user with an existing registration.  Below is a sample request payload:

```
{
    "clientDataJSON": "eyUyBg8Li8GH...",
    "authenticatorData": "o2M884Yt0a3B7...",
    "credentialId": "VGhpcyBpcyBh...",
    "signature": "OP84jBpcyB...
}
```


If successful, the response format is as follows:
```
{
    "id_token": "eyJ0eXA.2NDUxMjV9.5Od-8LjVM",
    "token_type": "Bearer",
    "access_token": "6ImNsb3VkSWRlbnRpdHlSZW",
    "expires_in": 604800
}
```