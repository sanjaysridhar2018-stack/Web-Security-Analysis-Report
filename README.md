# Web-Security-Analysis-Report
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
#### Impact
Sensitive information of a user including address and credit card details are vulnerable and can be accessed by anyone publicly
#### Solution
Validate ownership of an object before someone can access it by adding a layer of authorisation
### SQL Injection
#### Severity 
High
#### OWASP
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
1) The application responded with HTTP 200 OK
2) Valid JWT
3) Admin account access
   
#### Impact 
Unauthorised access to the admin account

#### Solution
Usage of parametrised SQL queries and dont concatenate the user input as an sql query 


