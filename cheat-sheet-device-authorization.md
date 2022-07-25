## Configuring the Janssen server

To use CURL commands and configure Janssen's Authorization server, you need to have an access token of "Config-API" (which is an RP of Jans-auth server). Configurations to the AS can be done **only** through "The Config-API client (RP)".

### 1. Obtaining an Access token
All commands to configure the AS are protected by an Access token. According to the use case, you must specify the `scope` for which the access token has been requested.

|  📝 Note: |
|:---|
| The client_id for Config-API is  `1800.c60b3b0c-7841-4898-8209-6d5b2305441f`. <br/>For the client_secret, contact your administrator.|


```
curl -u "client_inum:client_secret" https://<your.jans.server>/jans-auth/restv1/token -d  "grant_type=client_credentials&scope=put_scope_name_here
```
For e.g.: To modify a custom script, you need to request an access token using the scope `scope=https://jans.io/oauth/config/scripts.write`
```
curl -u "1800.c60b3b0c-7841-4898-8209-6d5b2305441f:put_config_api_client_secret_here" https://<your.jans.server>/jans-auth/restv1/token -d  "grant_type=client_credentials&scope=https://jans.io/oauth/config/scripts.write" 
```

***

### 2. Enable an authentication script
Steps:
1. Obtain a token, use scope `https://jans.io/oauth/config/scripts.write`
```
curl -u "1800.c60b3b0c-7841-4898-8209-6d5b2305441f:put_config_api_client_secret_here" https://<your.jans.server>/jans-auth/restv1/token -d  "grant_type=client_credentials&scope=https://jans.io/oauth/config/scripts.write" 
```
2. Enable the script
```
curl https://<your.jans.server>//jans-config-api/api/v1/config/scripts/name/name_of_the_script
```
Examples of `name_of_the_script` that comes Out of the box with the Janssen server.

| Name of the script |
|---|
| smpp  |
| otp |
| duo |
| fido2 |
| super_gluu |
| twilio_sms |
| smpp |
| otp |
| duo |
| fido2 |
| super_gluu |

***

### 3. Add scope to client
1. Obtain the pre-existing scopes of the client
    1. Obtain an Access Token with scope `https://jans.io/oauth/config/openid/clients.readonly`.
    ```
     curl -u "1800.c60b3b0c-7841-4898-8209-6d5b2305441f:put_config_api_client_secret_here" https://<your.jans.server>/jans-auth/restv1/token -d  "grant_type=client_credentials&scope=https://jans.io/oauth/config/openid/clients.readonly" 
    ```
   2. Obtain client information using:
    ```
     curl -X GET https://jans-ui.jans.io/jans-config-api/api/v1/openid/clients/client-s_inum_for_which_scope_to_be_added -H "Authorization: Bearer put_access_token_here"
    ``` 
   3. Notice the `scope` field. It is a space-seperated String of scope values e.g `"scope" : "openid user_name "`. To this, lets append the profile, so the scope attrib should now have value "openid user_name profile"`. This new value will be patched onto the client. 
   
    

1. Patch the client
	1. Obtain an Access Token with scope `https://jans.io/oauth/config/openid/clients.write`
	```
	curl -u "1800.c60b3b0c-7841-4898-8209-6d5b2305441f:put_config_api_client_secret_here" https://<your.jans.server>/jans-auth/restv1/token -d  "grant_type=client_credentials&scope=https://jans.io/oauth/config/openid/clients.write" 
	```
	2. Patch the new scope for the client 
	```
	curl -X PATCH -k -H 'Content-Type: application/json-patch+json' -i 'https://jans-ui.jans.io/jans-config-api/api/v1/openid/clients/61daeea0-9c12-4b95-b14c-c08765c63198' -H "Authorization: Bearer 8049c338-3043-40ab-805c-f14d9a24de4d" --data '[
	  {
		"op": "add",
		"path": "/scope",
		"value": "openid user_name profile"
	  }
	]'
	```
***	
### 4. Get grant_types for client

1. Obtain an Access Token with scope `https://jans.io/oauth/config/openid/clients.readonly`.
```
 curl -u "1800.c60b3b0c-7841-4898-8209-6d5b2305441f:put_config_api_client_secret_here" https://<your.jans.server>/jans-auth/restv1/token -d  "grant_type=client_credentials&scope=https://jans.io/oauth/config/openid/clients.readonly" 
```
2. Obtain client information using:
```
 curl -X GET https://jans-ui.jans.io/jans-config-api/api/v1/openid/clients/client-s_inum_for_which_grant_types_to_check -H "Authorization: Bearer put_access_token_here"
``` 
3. Notice the `grant_types` field in the response.  
   


### Using the Janssen server

## 1. OpenID Discovery endpoint / Well-known endpoint
```
curl https://jans-ui.jans.io/.well-known/openid-configuration
```
## 2. Client creation
Steps:
1. Download this [json file](https://raw.githubusercontent.com/JanssenProject/jans/main/jans-config-api/server/src/test/resources/feature/openid/clients/client.json), update the values and save it as client.json
1. Run curl command

```
curl -X POST https://my.jans.server/jans-auth/restv1/register -H "Content-Type: application/json"  -d @/some/directory/client.json
```

Further reading

## 2. Enable an authentication script
Steps:
1. Obtain a token
```
curl -u $TESTCLIENT:$TESTCLIENTSECRET https://<your.jans.server>/jans-auth/restv1/token -d  "grant_type=client_credentials&scope=https://jans.io/oauth/config/scripts.write" 

```


## 4. get grant_types for client
## 5. add OpenID scope and map to database attribute
