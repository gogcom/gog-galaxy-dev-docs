# OpenID

Galaxy supports user authentication using OpenID Connect.
As for now there is no support for OpenID Discovery so the connection between the issuer(GOG) and platform must be made explicitly.

## OpenID scope

You need to submit a ticket to our support so we can add OpenID scope to your game.

## PlayFab

1. Create a manual connection between PlayFab and GOG token provider. Setting this up through PlayFab will not work due to no support for OpenID Discovery through GOG.

Use can use this python script to do that:

```python
import requests
from requests.structures import CaseInsensitiveDict

url = "https://{titleid}.playfabapi.com/Admin/CreateOpenIdConnection"

headers = CaseInsensitiveDict()
headers["X-SecretKey"] = "{secret_key}" # taken from PlayFab's DevPortal
headers["Content-Type"] = "application/json"

clientID = "" # taken from GOG's DevPortal
clientSecret = "" # taken from GOG's DevPortal

# ask our support for JsonWebKeySet

data = """
{
 \"ClientId\": \"{client_id}\",
 \"ClientSecret\": \"{client_secret}\",
 \"ConnectionId\": \"GOG\",
 \"IgnoreNonce\": true,
 \"IssuerInformation\":
 {
 \"AuthorizationUrl\": \"https://auth.gog.com/auth\\",
 \"Issuer\": \"https://gog.com\\",
 \"JsonWebKeySet\":
 {
 },
 \"TokenUrl\": \"https://auth.gog.com/token\\"
 }
}
"""
resp = requests.post(url, headers=headers, data=data)
print(resp.status_code, resp.reason, resp.url)
print("Message:")
print(resp.text)
```

2. Request OpenID authorization feature to set a proper scope for the games's ClientID.
3. Setup config file, we used `GalaxyPeer.ini`

```
[DEFAULT]
openid = true
```

4. The JWT can be retrieved through `galaxy::api::Users()->GetIDToken()` after logging in through `galaxy::api::Users()->SignInGalaxy()`, which can then be used as the id token when login in through OpenId (the PlayFab plugin)
