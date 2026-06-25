# Web-Security-Analysis-Report
## Introduction
This is a penetration test report of the website Owasp Juice shop conducted on http://localhost:3000 which showcases hands on experience in vulnerability spotting in areas like DOM XSS, SQLi, BOLA, Information disclosure and showcases my expertise in using tools like Burp Suite for reconnaisance. 

These vulnerabilities were spotted both manually as well as using burp suite and highlights unauthorised access to execute sql commands as well as Javascript and privelege escalation.
Actual link: https://preview.owasp-juice.shop/#/
## Scope
The vulnerability analysis was done on the basis of 
1) exposed endpoints
2) Session and cookie management
3) Information disclosure
4) Client side security to santize the user input

## Methodology
The reconnaisance process was started by looking around the pages of the website and getting the requests stored on burp suite later at the same time which would help us in analysis later on. This was required to find exposed endpoints, possible oppurtunities for privelege escalation, to find stored cookies and JWTs which can be exploited. Then later on an account could be created to check if a JWT is stored and analysis could be made on it. The important requests are listed in a table below:

| Method | Endpoint | Purpose | Auth Cookies/JWT |
|--------|----------|---------|----------|
| Get | rest/product/1/reviews| get reviews of product 1 | Yes |
| Post | api/Feedbacks/ | post feedback | Yes |
| Get | /rest/user/whoami?fields=id,email | returns info about authenticated user| Yes |
| Get | /rest/captcha | captcha verification| Yes |
| Get | /rest/basket/6 | View basket | Yes |
| Get | /api/Addresss/7 | address of user| Yes |
| Get | /api/Deliverys/1 | deliveriess of user | Yes | 
| Get | /api/cards/1 | credit card details | Yes |
| Get | /rest/admin/application-configuration | Not sure | Yes | 
| Post | /rest/user/login | login | Yes | 
| Post | /socket.io/?EIO=4&transport=polling&t=Pxzw6k8&sid=7LwQeY3JyRYCkqr1AAAI | Change language | Yes |

This gave us huge insights since 
1) basket, addresss, deliverys, cards endpoints have a possibility of being exploited by BOLA -critical
2) the rest/admin gives potential access to an admin endpoint - critical 
3) /user/login has a post query which gives us the chance to perform SQLi and potentially bypass it if vulnerability exists- critical 
4) /captcha suggests that maybe there is a vulnerability suggesting that captcha can be bypassed - low
5) the lang paramter is a very critical vulnerability; it not only takes the languages available like en or fr but since the value of the parameter is not authorised, there is a risk of simply accepting what the user has provided. So instead of lang = en the user can write lang = php and load a payload- critical but not discussed
6) session management: the jwt token is stored twice, once along with the cookies and once with the authorisation: bearer header which suggests that maybe either omitting one of those or possibly even both of those can give us some insights-moderate
7) The search bar present in the website gives us an oppurunity to maybe perform a cross side script if the user input is not sanitized thus allowing us to enter a payload into the DOM. - critical
8) The presence of a password parameter allows us to try and password hash using a common password wordlist IF the site does not set a limit to the number of post requests that can be provided thus enabling us to monitor the security


## Risk Summary

| ID | Vulnerability | Severity | OWASP Top 10 (2021) | Risk |
|----|---------------|----------|----------------------|------|
| F-01 | Broken Object Level Authorization (BOLA) | High | A01 – Broken Access Control | High |
| F-02 | SQL Injection (SQLi) | High | A03 – Injection | High |
| F-03 | DOM-Based Cross-Site Scripting (XSS) | High | A03 – Injection | High |
| F-04 | Missing Brute Force Protection | Medium | A07 – Identification and Authentication Failures | Medium |
| F-05 | Improper Authentication Enforcement | Medium | A07 – Identification and Authentication Failures | Medium |
## Findings 

### Broken Level Object Authorisation(BOLA)

#### Severity
High

#### OWASP 
1) OWASP API Top 10: API1 - Broken Object Level Authorization
2) OWASP Top 10: A01 - Broken Access Control
   
#### End points
1) GET /api/basket/id
2) GET /api/addresss/id
3) GET /api/deliverys/id
4) GET /api/card/id
   
#### Description
The user can access another user's resources and personal details without authorisation by just changing the ID parameter

#### Steps
1) Login as user
2) intercept the get request in burp suite since few api requests are sent in the backend 
3) send the request in proxy to repeater
4) change the product id and observe the results in the response tab
   
#### Proof 
<img width="725" height="389" alt="Screenshot 2026-06-25 142058" src="https://github.com/user-attachments/assets/9486c096-124a-49f1-b99c-4791a3836421" />
In the given screenshot, observe line 1. The /basket/6 has been changed to /basket/5 allowing me to access someone elses cart

#### Impact
Sensitive information of a user including address and credit card details are vulnerable and can be accessed by anyone publicly

#### Solution
Validate ownership of an object before someone can access it by adding a layer of authorisation

### SQL Injection

#### Severity 
High
#### OWASP
OWASP Top 10 2021: A03 – Injection
OWASP API Security Top 10 2023: API8 – Security Misconfiguration

#### Endpoints
POST /rest/user/login

#### Description 
The site was checked if it had an SQLi vulnerability at the authentication end point. Since the application failed to secure the end point, an SQL payload was crafted and sent to bypass the password parameter. This helped in bypassing the authentication end point and accessing the admin account

#### Steps
1) At the login page enter ' as the username parameter to see how it responds and if its vulnerable to SQLi 
2) If its vulnerable craft a payload by rendering the password parameter as a comment such that its not asked for but at the same time make sure the authentication is bypassed by always returning a true statement
3) The payload crafted for this was " OR '1'= '1' --
4) The given payload works since " is used to close the username query and since 1=1 is always true and the or statement returns true if any one is true, a true statement is passed
5) -- is written at the end to render the password parameter useless by treating it as a comment

#### Proof of concept

<img width="917" height="366" alt="Screenshot 2026-06-25 150928" src="https://github.com/user-attachments/assets/a72c607c-c061-4043-ab69-4663264fa4ff" />

1) The application responded with HTTP 200 OK
2) Valid JWT
3) Admin account access

<img width="938" height="370" alt="Screenshot 2026-06-25 150951" src="https://github.com/user-attachments/assets/0af05d69-9977-451b-bc59-44330732743d" />

#### Impact 
Unauthorised access to the admin account

#### Solution
Usage of parametrised SQL queries and dont concatenate the user input as an sql query 

### DOM based cross side scripting 

#### Severity 
High

#### Owasp 
OWASP Top 10 2021: A03 – Injection

#### Description
The dom based vulnerability was found in the search bar since the application again failed to sanitize the user input before inserting it in the dom. Thus the user is allowed to inject a malicious payload in the javascript 

#### Area of vulnerability
product search bar

#### Proof

<img width="955" height="388" alt="Screenshot 2026-06-25 162135" src="https://github.com/user-attachments/assets/8401f772-6916-432e-b9f1-654f0b87c019" />

"<iframe src="javascript:alert(`xss`)">" was the payload that was injected in the product search bar which caused the alert query to be added to the javascript causing an alert box could appear. This proves that the dom is vulnerable and if a malicious payload was crafted, it could be inserted into the DOM easily. In this context, iframe was used to load the JS url rather than script since the website had added <script> to its blocklist which allows us to take advantage of this less obvious XSS command.

<img width="953" height="400" alt="Screenshot 2026-06-25 162126" src="https://github.com/user-attachments/assets/9910b02b-f79f-4fa2-a649-5f146330ef77" />

#### Impact
1) crafted payloads can redirect users to malicious websites
2) steal session tokens and cookies
3) phising attacks
#### Solution
1) Sanitize the user query before adding it to the DOM
2) Awareness should be increased to add alternative commands to the blockist like how <script> was added so there would be no room for penetration

### Brute Password Forcing
#### Severity
Medium
#### Owasp Mapping
OWASP Top 10 2021: A07 – Identification and Authentication Failures
OWASP API Security Top 10 2023: API2 – Broken Authentication
#### Description
The very first thing when I came across the password field was whether it can withstand brute force attacks. That is, to check if there is a limit on the number of post requests you can send to prevent password hashing. However there was no such limitation making it prone to loading a password word list on burp suite intruder and trying to force the password.
#### Affected End point
POST /rest/user/login
#### Proof

<img width="955" height="400" alt="Screenshot 2026-06-25 161138" src="https://github.com/user-attachments/assets/b1d0a399-d939-4bf0-9192-75055069bf53" />

A sniper attack was configured by passing the request to the intruder tab in burp suite and was loaded with a password.txt folder to try and brute force the password. Despite hundreds of attempts, no HTTP 429 Too many requests error arose.

<img width="920" height="455" alt="Screenshot 2026-06-25 161406" src="https://github.com/user-attachments/assets/a87cddec-4832-4315-b45f-44f078d72361" />

#### Impact:
Automated password guessing leading to unauthorised access

#### Solution:
Require captcha which cannot be bypassed or a secure multi factor authorisation system or limit the number of failed requests that can be sent at a time

### Improper Authorisation Enforcement
#### Severity 
Moderate
#### Owasp 
A07 – Identification and Authentication Failures
#### Exposed endpoint
/rest/admin/application-configuration
#### Proof
While investigating the endpoint since it is present in the admin directory, it was peculiar that the JWT was stored in 2 seperate locations 
1) the authorisation: bearer header
2) cookies
So I tried deleting the header first which gave the following result:

<img width="775" height="363" alt="Screenshot 2026-06-25 144819" src="https://github.com/user-attachments/assets/9ae29ea1-de32-4701-94c2-e8cb60142bf3" />

Then i tried deleting only the cookie which gave a similar result but on deleting both, it exposed this admin endpoint which shows the lack of security of the site 

<img width="732" height="289" alt="image" src="https://github.com/user-attachments/assets/c30ede27-a1e7-4759-b965-0fbafc52f66d" />

#### Impact
Attacker can access the resources which only the admin is meant to access

#### Remediation
Validate JWTs before processing requests


## Conclusion
The pentesting of the OWASP Juice shop led to 5 critical - medium vulnerabilities which have been deeply analysed proving that they possess a strong threat. Immediate remediations must be done to protect the OWASP Juice shop site from being exploited in the future





