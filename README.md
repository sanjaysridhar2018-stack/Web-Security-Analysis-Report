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

