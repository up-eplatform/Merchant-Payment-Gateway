# Web Merchant Integration Guide

# INTRODUCTION 

This guide provides details on how the integration to the Unified Payments payment gateway should be implemented. The payment gateway provides for payment using the followingschemes: 
* PayAttitude (Pay with Phone number) 
* Visa 
* MasterCard and 
* Verve 

This data exchange format for this implementation is JSON.
Do not hesitate to engage us through eplatform@up-ng.com for  support regarding this implementation




# Look and Feel
To have a feel of our payment page, fund your mobile phone line on https://www.payarena.com 
On the home page:
* Select Airtime Recharge and select the Telco you prefer On the Telco’s page
* Enter your phone number, amount and email address and submit the form On the checkout page
* Click on the “Click here to pay” button  The next page displayed to you is the payment page. 
* On this page, select Pay with Phone to pay with your phone number (PayAttitude) or  Pay with Card to pay with your debit card (Mastercard, Visa or Verve)
* For Pay with phone, you need to download from Google play store or iOS App store, install and setup the PayAttitude Digital Mobile App on your mobile phone. When you enter your phone number, a notification will be sent to the PayAttitude App on your phone. 
* Enter your PIN on the app to authorize the transaction. 
* For Pay with Card, enter your card details and submit the form. On the ACS page, enter the OTP you receive from your issuer/bank and submit the form
The transaction confirmation page will be displayed to you after the authentication is  completed.

## INTEGRATION PROCEDURE

* GO TO https://test.payarena.com/prospectivemerchants on your browser
* Fill in details in the form that will be presented to you.
* Submit the form
* Test Configuration details will be forwarded to you via the email you provided. This will include the following: 
   Merchant ID
   Cryptographic Key
* The Merchant ID and Cryptographic key are used to identify a Webshop on UP MPI each time a Webshop sends a payment request, i.e. a create order request to UP MPI.
* Review this documentation and commence implementation
* Review UAT documentation and ensure that all UAT requirements are met on merchant site
* System Integration Test and User Acceptance Test with Unified Payments team
* On successful UAT, Unified Payments team will initiate Go-live process and provide production parameters
* Apply production parameters and confirm successful pilot


![TRANSACTION PROCESS FLOW](https://github.com/up-eplatform/Merchant-Payment-Gateway/blob/8eedd285596cb7eb787869737e5957a61a7b3728/img/transaction%20flow.png)

![TRANSACTION PROCESS FLOW](https://github.com/up-eplatform/Merchant-Payment-Gateway/blob/8eedd285596cb7eb787869737e5957a61a7b3728/img/process%20flow.png)

## Appendix A


* On checkout, the webshop saves the transaction in the webshop’s database sets the status  field as pending 
* Set in your request header, accept to application/json i.e. Accept:application/json 
* The CreateOrder post request is made to the URLhttps://test.payarena.com/MERCHANTID.The Merchant ID is to be replaced with your merchant ID sent to you via email. 
* The post data should be in the json format as below 

```json
{
  "amount" : 400, 
  "currency" : 566, 
  "description" : 
  "Description of product or service",  
  "returnUrl" : "http://testurl.com/response/", 
  "secretKey" : "YOUR-SECRET-KEY", 
  "fee": 0 
  } 
  ```
* Find the details of each field in table1.0. 6. Replace the secret key with the cryptographic key sent to you via email

* The currency should be the ISO 4217 number of the transactioncurrency 
* The amount displayed to the customer must be the same as the amount sent to the  payment gateway
* The returnURL is the URL the UP MPI redirects to after the payment is complete 
* The transaction ID from your application, IP address of the customer’s computer or any extra parameter can be included in the description field. For example **"description":"Description of service^REF:123456(IP:41.8.29.132)"**

**Note:** You are expected to capture the customer’s location, log it on the web shop’s database and pass the customer’s IP address in the description field as above Also ensure that input validation on your website is properly enforced. Input validation implies validation of user details especially phone number and email address.

**Table 1.0**
|Field | Type |Max | Length |
| --- | --- |--- | --- |
| Amount| Decimal (18,2) |18 |  |
| Currency| Integer |5 | ISO 4217 number |
| Description| VARCHAR |150 |  |
| returnURL| VARCHAR |150 |  |
| Secret Key| VARCHAR(255) |500 |  |
| Fee Decimal| Decimal (18,2) |18 |  |


## Appendix B
On receipt of a create order request, the UP MPI saves thetransaction  details, i.e. the amount, description, secret, sent in the payload.

The UP MPI generates a session with the received transaction details  and generates a unique ID that identifies this transaction. 
This is called  the transaction ID. The MPI sends the transaction ID as a response to  the create order request.

This transaction ID is tied to the transaction received by the MPI and so in the subsequent requests for this transaction, to make payment  and get the transaction status, WebShop is expected to send this  transaction id to UP MPI.

UP MPI will use this ID to query the details of this transaction and  provide the required response.


## Appendix C
After the UP MPI responds with the transaction ID, you are expected to
* 1. Update the transaction details in the web shop’s database with the transaction ID in the response to the create order request
* 2. Redirect the customer to UP MPI to make payment using HTTP GET protocol passing the Transaction Id. e.g. https://test.payarena.com /2435647. 
Where 2435647 is a transaction ID.


## Appendix D
The 3DSecure Provider means any party that provides 3DSecure services. This could be the scheme or any third party that assists to provide 3DSecure services for the Issuer.

## Appendix E
* 1. After the payment is made, UP MPI will redirect back to the app via thereturnUrl, sent when initiating the transaction, using a HTTP POST. 
* 2. UP MPI will pass the parameters Transaction Id, status and approved in the POST  request to the returnURL. E.g.

```json
{

"trxId": "54321", 
"status": "APPROVED", 
"approved": "true"

} 
```
Note: trxId is the transaction’s ID on the UP MPI status is the status of the transaction, this could be either APPROVED or  DECLINED approved is a flag of value true for approved transaction status or false for  declined transaction status(see table 2.0 for transaction status and description of status) 

* 3. You must query the UP MPI server to obtain the correct status of the transaction. 
* 4. To query the status of a transaction, send a get request to the UP MPI server e.g.  https://test.payarena.com /Status/transactionId. Where transactionId is the  transaction ID sent as a response after the order was created.
* 5. The payment gateway will respond with the complete detail of the transaction status  in a json response. Below is a sample:

```json
{
 "Order Id": "54321",
 "Amount":"1500.00", 
 "Description":"Native Shirt REF:123456  (IP:41.8.29.132)", "Convenience Fee":"0.00", "Currency":"566", "Status":"APPROVED", "Card  Holder":null, "PAN":"987650XXXX4321",           "Scheme":"VISA", 
 "StatusDescription":"APPROVED",  
 "Approval Code":"012345", 
 "TranDateTime":"12/10/2018 10:43:28"
 }
 ```
* 6. Store the raw/exact json response in the web shop’s database. This will be required  for the resolution of disputes that mayarise 
* 7. Extract the information from the json response and make this visible on the  transaction log table on the admin portal and customer transaction history viewon  the webshop
* 8. Table 1.1 contains the details of the data in the response 
* 9. Update the status of the transaction on the web shop’s database with the status of in  the json response 
* 10. Display a receipt to the customer showing the details of response received and send  email as well to the customer. It is important to show the details of the response  received in the receipt. 

**Note:** The transaction confirmation received could be a decline or approval. The admin  portal and the customer history should have a view page that shows the complete  response for each transaction detail including the raw json response

**Table1.0**
|Field |Type | Description |
| --- | --- |--- |
| OrderId| Integer |This is the same as the transactionId | 
| Description| String |  Description of the product/service sent to  the payment gateway| 
| Amount| Decimal(18,2) |The transaction amount|  
| TranDateTime| String |Transaction Date and time|  
| PAN| String |Masked Pan of customer| 
| Cardholder| String | Name of Cardholder| 
| ConvenienceFee| Decimal(18,2) | | 
| Currency| Integer | ISO 4217 number of the transaction  currency|
| ApprovalCode| String | The Approval Code only exists for  approved transactions| 
| Scheme| String | Scheme used by card holder|
| Status| String | Status of the transaction usually approved  or declined|
| StatusDescription| String | Description of the status|



**Table 2.0**

**Transaction Status and Description**

|Status |Description | 
| --- | --- |
| Approved|A transaction is approved when the customer is successfully debited and value for transaction is received.|
| Cancelled|A transaction is cancelled when the customer decides to not enter payment details and returns back to the merchant site.| 
| Declined|A transaction is declined when one of the following happens:Unsucccessful authentication, Unsuccessful authorization, System error|
| Initiated|A transaction is initiated when the customer abandons a transaction.|

 






 


   
    

## Contact
Created by [@eplatform(eplatform@up-ng.com) - feel free to contact us for clerifications.
