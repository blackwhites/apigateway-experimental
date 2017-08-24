# Getting started with OAuth 2.0 and the new API Gateway

## Overview
The purpose of this guide is to show the user how to setup OAuth in the new API Connect Toolkit.

## Assumptions
The user has succesfully setup the API Connect Toolkit and can test their API's (the README.md file in this project contains the details).

## Steps

### Enable Experimental Mode

1. Turn on "experimental mode" in APIC Toolkit 

> ```apic config:set datapower-api-gateway-experimental=true```

2. Set the oauth redirect URI to the localhost so that you may test on your laptop

> ```apic config:set oauth-redirect-uri=https://localhost``` 

3. Set the oauth client type to confidential or public

> ```apic config:set oauth-client-type=confidential``` 

---

### Edit the configuration

> ``` apic edit```

1. Sign in or Register at the login screen

2. Navigate to the Security Section (Click ```Security```)

![Image of Datapower Reboot](/images/oauth20/security_section.png)

3. In the Security Section, ensure OAuth Providers is selected and click the Add button on OAuth Providers.

![Image of Datapower Reboot](/images/oauth20/security-oauth.png)

4. The ```Create OAuth Provider``` screen will appear after clicking the Add button

![Image of Datapower Reboot](/images/oauth20/oauth_provider.png)

5. Provide a name say, ```first-oauth-provider``` and give a Base path, say ```/```.

Note: Base path must be unique. No two OAuth Provider objects can have the same base path. This base path will be used to auto generate an API. ( See next steps)

6.	Click save. You can click on the name again to see the details of the API and change any default values if you need to.  The following are the default values:

    1.	Provider type is “APIConnect”, which means APIC will generate the access tokens.
    1.	Scopes: Default is “scope1”.
    1.	Grants: “Application”, "Implicit" and “Access Code” grant types are enabled by default. "Application" grant only works with confidential clients.
    1.  Client Types: confidential enabled by default
    1.	Authorize Endpoint: /oauth2/authorize
    1.	Token Endpoint: /oauth2/token
    1.  Token Introspection: default disabled
    1.  APIC Token Secret: oauth-default-key.  This corresponds to the shared secret key object created in DataPower. Toolkit creates a default SS key named “oauth-default-key”. This is not editable from UI at this point. If you need to change this, you have to go to the “definitions” folder in the toolkit and update the yaml, but you should also ensure to create the key file and object in DataPower.
    1.	Access Token TTL: default 3600 seconds
    1.	Maximum consent TTL: default 7200 seconds
    1.  Refresh Token: default disabled

7. Start the gateway if it is not already running.

![Image of Datapower Reboot](/images/oauth20/running.png)

8. Once the gateway starts, it will detect that an OAuth Provider Settings object exists and it will auto generate a corresponding API.

![Image of Datapower Reboot](/images/oauth20/provider_exists.png)

This API is like any other API, but in the Assemble action it has an OAuth-generate action that references to the OAuth Provider Settings object.

![Image of Datapower Reboot](/images/oauth20/oauth-generated.png)

Note: Some default parameters and security definitions/requirements are added to this API. But you can edit this API like any other normal API. For example, update the API security definitions or requirements, request and response parameters or add policies to the assembly that needs to be executed before the token generation as per your needs. But you can’t change the base path, authorize endpoint and token endpoint directly in the API. 

---

### Testing

Now you can send a request to test the Client credentials grant type. Use postman or curl


#### Curl
```
curl -k -X POST \
  --url 'https://127.0.0.1:4002/oauth2/token' \
  --header 'accept: application/json' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'client_id=default&client_secret=SECRET&grant_type=client_credentials&scope=scope1'
  ```

Note: A default client is created by the toolkit. Its client_id is “default” and client_secret is “SECRET” 
You should get an access token back:

```
{
  "token_type": "bearer",
  "access_token": "AAIHZGVmYXVsdMeldWS6j1KEm0pi0FRbdRm-aOq9y3nt2l3azyZO9XaK4xmiB6YgY_6GtVBHSo9yiU4F3VD2Qlp_UR7_d8OUew0",
  "scope": "scope1",
  "expires_in": 3600,
  "consented_on": 1498852931
}
```

#### Postman

The following instructions are for retrieving the token using the Postman App, not the Postman Chrome App (installed via the Chrome App Store). Postman can be downloaded from [Postman WebSite](https://www.getpostman.com/)

### Disable SSL certificate verification

1. Click the "wrench" at the upper right corner of Postman, select ```Settings``` when the menu options appear

![Image of Datapower Reboot](/images/oauth20/settings.png)

2. The Settings Window will appear after Selecting ```Settings```, make sure to select the ```General``` tab/section.  Clock the ```X``` at the upper right corner to save and dismiss the screen.

![Image of Datapower Reboot](/images/oauth20/cert-verify.png)

### Create a Request
From the Postman Screen

![Image of Datapower Reboot](/images/oauth20/request.png)

1. Add a new Tab if needed by pressing the ```+``` tab.

2. Change the request from ```GET``` to a ```POST```

3. Add the URL ```https://127.0.0.1:4002/oauth2/token``` at ```Enter request URL```

4. Click the ```Headers``` tab/button, click ```Bulk Edit``` and enter the following value:

```
Content-Type:application/x-www-form-urlencoded
```

The screen should look like this:

![Image of Datapower Reboot](/images/oauth20/post_header.png)

5. Next Click the ```Body``` tab/button; then click the radio button ```x-www-form-urlencoded```.   

6. Click ```Bulk Edit``` and enter the following value:

```
client_id:default
client_secret:SECRET
grant_type:client_credentials
scope:scope1
```

The screen should look like this
![Image of Datapower Reboot](/images/oauth20/post_body.png)

7. Press the Send button when you want to retrieve the token, check the ```Response``` Window for the token

![Image of Datapower Reboot](/images/oauth20/result.png)

## Additional Examples

The examples below are additional features of the oauth provider.

### Implicit Grant

1. Enable the Implicit grant in the oauth provider

![Image of Datapower Reboot](/images/oauth20/implicit.png)

2. Open the api ```OAuth API for the provider of the first-oauth-provider``` in the api tab

3. Navigate to the ```basicAuth``` security definition by clicking on ```Security Defintions``` on the left side and scroll down

4. Change the URL protocol to ```http``` or copy paste the URL below

URL: ```http://httpbin.org/basic-auth/testuser/testpw```

![Image of Datapower Reboot](/images/oauth20/basicAuth.png)

5. Now you can send a request to test the Implicit grant type. Use postman or curl

#### Curl
```
curl -k -X GET \
  --url 'https://127.0.0.1:4002/oauth2/authorize?client_id=default&response_type=token&redirect_uri=https%3A%2F%2Flocalhost&scope=scope1' \
  --header 'authorization: Basic dGVzdHVzZXI6dGVzdHB3'
  ```

Note: The Implicit grant returns a 302 Found redirect with URL fragment that contains the access token.

```
Location -->  https://localhost#access_token=AAIHZGVmYXVsdDkYYyIik_qRxAUoxqWwlS0xp9KVOLPOxOg9ctfNYSN2WVdxiXDiwfdbVC5DRDowoMhNcijhqpduu8VQMSLXa5opNI9s_AaDG8jSY32P4gUj&expires_in=3600&token_type=bearer&scope=scope1&consented_on=1503413966
```

#### Postman

1. Add a new Tab if needed by pressing the ```+``` tab.

2. Add the URL ```https://127.0.0.1:4002/oauth2/authorize``` at ```Enter request URL```

3. Click Params then ```Bulk Edit``` and enter the following values:

```
client_id:default
response_type:token
redirect_uri:https://localhost
scope:scope1
```

4. Click the Authorization tab/button and select ```Basic Auth``` as the type

5. Enter the following values in the ```Username``` and ```Password``` fields.

```
Username: testuser
Password: testpw
```

6. Click the ```Update Request``` button

![Image of Datapower Reboot](/images/oauth20/post_implicit_request.png)

7. Press the Send button when you want to retrieve the token, check the ```Header``` tab/button in the ```Response``` Window for the token

![Image of Datapower Reboot](/images/oauth20/post_implicit_response.png)

### Authorization Grant

1. Enable the Authorization grant in the oauth provider

![Image of Datapower Reboot](/images/oauth20/access_code.png)

2. Open the api ```OAuth API for the provider of the first-oauth-provider``` in the api tab

3. Navigate to the ```basicAuth``` security definition by clicking on ```Security Defintions``` on the left side and scroll down

4. Change the URL protocol to ```http``` or copy paste the URL below

URL: ```http://httpbin.org/basic-auth/testuser/testpw```

![Image of Datapower Reboot](/images/oauth20/basicAuth.png)

5. Now you can send a request to test the Authorization grant type. Use postman or curl

#### Curl
```
curl -k -X GET \
  --url 'https://127.0.0.1:4002/oauth2/authorize?client_id=default&response_type=code&redirect_uri=https%3A%2F%2Flocalhost&scope=scope1' \
  --header 'authorization: Basic dGVzdHVzZXI6dGVzdHB3'
  ```

Note: The Authorization grant returns a 302 Found redirect with URL query that contains the authorization code. 

```
Location -->  https://localhost?code=AAIkg4ddjCQO0A8o0C5Z95L9oNaMK2uTJnUkTI9KbAB5nAG-MV5Thu-h2OTP4rCwvT1PSrwNl2bc0mySlfmLCguaffdKyebnYYECJRNSMACdhg
```

Note: Another request will need to be made to obtain the access token so copy the code for the next request.

```
curl -k -X POST \
  --url 'https://127.0.0.1:4002/oauth2/token' \
  --header 'accept: application/json' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'client_id=default&client_secret=SECRET&grant_type=authorization_code&scope=scope1&redirect_uri=https%3A%2F%2Flocalhost&code=AAIkg4ddjCQO0A8o0C5Z95L9oNaMK2uTJnUkTI9KbAB5nAG-MV5Thu-h2OTP4rCwvT1PSrwNl2bc0mySlfmLCguaffdKyebnYYECJRNSMACdhg'
  ```
  
Note: A refresh token was returned because Authorization and Password grants are capable of returning a refresh token. This will only occur if refresh tokens are enable in oauth provider

```
{
  "token_type":"bearer",
  "access_token":"AAIHZGVmYXVsdKZRcBr6cX2b1X2tYSRoytXkFyEQSdfuX5zewu2QwDJ9-W54aWUHyuPB3sIZk-MrvIYotwR29PDv8TnBHeYPGAn-sWE5vlpHjhIdttJQOsQD",
  "scope":"scope1",
  "expires_in":3600,
  "consented_on":1503415347,
  "refresh_token":"AAJgjKiKAPOnaDJeEIFDplFK5Ct2244Rz_SyvmpOzX9mEVrpWmqyAAPgF_ijcLhZ48X5zp4qREQx-uSRFc_1qfZa0Fn2iKqpxEjRivr3Rb1RXg",
  "refresh_token_expires_in":5400
}
```

#### Postman

1. Use the two postman requests mentioned above for the authorize and token endpoint

2. Edit the GET request for the ```authorize``` endpoint

Note: ```response_type``` was changed to ```code```

![Image of Datapower Reboot](/images/oauth20/post_az_authorize.png)

3. Press the Send button when you want to retrieve the authorization code, check the ```Header``` tab/button in the ```Response``` Window for the authorization code 

4. Copy the ```authorization code```

5. Edit the POST request for the ```token``` endpoint

Note: ```grant_type``` was change to ```authorization_code```.
Note: ```redirect_uri``` was added
Note: ```code``` was added

![Image of Datapower Reboot](/images/oauth20/post_az_token.png)

6. Press the Send button when you want to retrieve the token, check the ```Response``` Window for the token

![Image of Datapower Reboot](/images/oauth20/post_az_response.png)

### Refresh token

1. Enable ```Refresh Token``` in the oauth provider

Note: ```Refresh token limit``` is default 10 and ```Time to live``` is 5400 seconds

![Image of Datapower Reboot](/images/oauth20/refresh_token.png)

2. Now you can send a request to test the Refresh Tokens. Use postman or curl

Note: Use the Authorization Grant above to obtain a request token

#### Curl

```
curl -k -X POST \
  --url 'https://127.0.0.1:4002/oauth2/token' \
  --header 'accept: application/json' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'client_id=default&client_secret=SECRET&grant_type=refresh_token&scope=scope1&refresh_token=AAJgjKiKAPOnaDJeEIFDplFK5Ct2244Rz_SyvmpOzX9mEVrpWmqyAAPgF_ijcLhZ48X5zp4qREQx-uSRFc_1qfZa0Fn2iKqpxEjRivr3Rb1RXg'
  ```
  
Note: A refresh token will be returned until the ```Refresh token limit``` is reached

```
{
  "token_type":"bearer",
  "access_token":"AAIHZGVmYXVsdCqQA2xcWLfQhSeA_UwGiNovHXD_Yb5hhn5y5XrkaU2GSwrEU1m_rZMhTGj4bLTsJn5f7XDN69sHx4W5GByKU3jj15aw94f35evlqpKFEAKs",
  "scope":"scope1",
  "expires_in":3600,
  "consented_on":1503416788,
  "refresh_token":"AAKIgZcDKcdJ9wI4NvAYlKWoQrLFhfCjuKtN7TTMOGfGgWYjcSRs7IO73pE_ZHG4esLbix5Sev7ANhNgK5tDf9qI67eMhAcLpERpP0sy18vH-w",
  "refresh_token_expires_in":5400
}
```

#### Postman

1. Edit the POST request for the ```token``` endpoint

Note: ```grant_type``` was change to ```refresh_token```.
Note: ```refresh_token``` was added

![Image of Datapower Reboot](/images/oauth20/post_refresh_token_request.png)

2. Press the Send button when you want to retrieve the token, check the ```Response``` Window for the token

![Image of Datapower Reboot](/images/oauth20/post_refresh_token_response.png)

### Token Introspection

1. Enable ```Introspection``` in the oauth provider

Note: ```APIC Introspection Endpoint``` is default /oauth/introspect
Note: The additional path added in the ```OAuth API for the provider of the first-oauth-provider```

![Image of Datapower Reboot](/images/oauth20/token_introspection.png)

2. Now you can send a request to test the Introspection. Use postman or curl

Note: Use one of the grants above to obtain a token

#### Curl

```
curl -k -X POST \
  --url 'https://127.0.0.1:4002/oauth2/introspect' \
  --header 'accept: application/json' \
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'client_id=default&client_secret=SECRET&token_type_hint=access_token&token=AAIHZGVmYXVsdOhFH4OIGXXMLAM-UWWh5F2LivKAApdLODlxtLSnIlQQaUemu4TzhLmqaAF9hok-Fzwc368rLzI6-KygnPZNmP1MgWFpg0MT3N32lTnARkC6'
  ```
  
Note: The contents of the token are returned in the response body

```
{
  "active":true,
  "scope":"scope1",
  "client_id":"default",
  "username":"testuser",
  "token_type":"bearer",
  "ttl":3215,
  "exp":1503420603,
  "expstr":"2017-08-22T16:50:03Z",
  "iat":1503417003,
  "nbf":1503417003,
  "nbfstr":"2017-08-22T15:50:03Z",
  "consented_on":1503417003,
  "consented_on_str":"2059-08-22T15:50:03Z"
}
```

#### Postman

1. Add a new Tab if needed by pressing the ```+``` tab.

2. Change the request from ```GET``` to a ```POST```

3. Add the URL ```https://127.0.0.1:4002/oauth2/introspect``` at ```Enter request URL```

4. Click the ```Headers``` tab/button, click ```Bulk Edit``` and enter the following value:

```
Content-Type:application/x-www-form-urlencoded
```

The screen should look like this:

![Image of Datapower Reboot](/images/oauth20/post_introspect_header.png)

5. Next Click the ```Body``` tab/button; then click the radio button ```x-www-form-urlencoded```.   

6. Click ```Bulk Edit``` and enter the following values:

Note: Make sure to copy paste a token from one of previous requests

```
client_id:default
client_secret:SECRET
token_type_hint:access_token
token:AAIHZGVmYXVsdOhFH4OIGXXMLAM-UWWh5F2LivKAApdLODlxtLSnIlQQaUemu4TzhLmqaAF9hok-Fzwc368rLzI6-KygnPZNmP1MgWFpg0MT3N32lTnARkC6
```

The screen should look like this
![Image of Datapower Reboot](/images/oauth20/post_introspect_body.png)

7. Press the Send button when you want to introspect the token, check the ```Response``` Window for the result

![Image of Datapower Reboot](/images/oauth20/introspect_result.png)
