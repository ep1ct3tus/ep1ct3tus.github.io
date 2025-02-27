+++
date = '2025-02-01T18:10:20+02:00'
draft = false
title = 'Apple Employees Data Exposure'
+++

###  Motivation
After reading [We Hacked Apple for 3 Months: Here’s What We Found](https://samcurry.net/hacking-apple) by Sam Curry
I felt motivated to search for more bugs and security flaws on Apple

I started collecting apple.com subdomains and began searching for odd or suspicious responses. 
Then I discovered that a subdomain on apple.com had a 404 not found error.

 ```helpcentral-xxxxxx.apple.com```

Further investigation revealed that it is the ServiceNow Knowledge Management Platform-hosted CentralStation system knowledge base portal for Apple employees.

### ServiceNow Knowledge Management: What is it?
>The ServiceNow platform has a feature called ServiceNow Knowledge Management that lets businesses produce, organize, and distribute knowledge articles to enhance customer service and give users the freedom to solve problems on their own.It is an essential part of 
the Customer Service Management (CSM) and IT Service Management (ITSM) solutions offered by ServiceNow.
Through easy access to information, it is intended to assist organizations in decreasing the frequency of repetitive inquiries, speeding up resolution times, and improving the user experience overall. It is extensively utilized in customer service, HR, IT support, 
and other divisions where information exchange is essential to operational effectiveness.
Users can find information about self-help, troubleshooting, and task resolution in articles found in ServiceNow Knowledge Management knowledge bases.

I came across an [article](https://medium.com/@th3g3nt3l/multiple-information-exposed-due-to-misconfigured-service-now-itsm-instances-de7a303ebd56) by researcher @Th3G3nt3lman that revealed details about improperly configured Service Now ITSM instances.

Users can access knowledge base articles with links like this:

>https://{company-subdomain}.{company-hostname|service-now.com}/kb_view_customer.do?sysparm_article=KB00xxxx

The ```sysparm_article``` parameter can be brute-forced with numeric IDs such as ```KB001000, KB001001, KB001002,``` and so on to retrieve knowledge base articles.

In this case, the articles are located at ```https://helpcentral-xxxxx.apple.com/kb_view_customer.do?sysparm_article=KB00XXXXX```  where XXXXX may have been ```KB0010000``` to ```KB0025000```.

After searching through thousands of Knowledge Base articles and hours of relentless bruteforcing with Burpsuite's Intruder, I finally found some articles that were revealing sensitive information and PII


>PII and email address from Apple’s internal team

![email](/img/apple-leak-1.png)

>Discovered a list of issues/bugs associated with different platforms and apps. 

![bugs](/img/apple-leak-2.png)

>App pilot testers 

![testers](/img/apple-leak-3.png)

>Attachments to the articles in the knowledge base

![attachments](/img/apple-leak-4.png)

### Timeline:
- 11 Jul 2021: I compiled and reported my findings to the Apple Security Team. 
- 11 Jul 2021: Apple Security Team’s initial response.
- 13 Jul 2021: Apple Security Team addressed this issue.
- 13 Aug 2021: Apple Security Team resolved this problem and modified the system.
- 13 Aug 2021: I confirmed that the Apple Security Team had resolved this issue.
- 31 Aug 2021: Apple Security Team credited me on their security credit page and awarded me $XX,XXX for this submission.

![bounty](/img/apple-leak-5.png)
