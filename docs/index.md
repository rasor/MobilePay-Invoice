---
layout: default
---


## <a name="overview"/> Overview

Billing your customers with MobilePay Invoice is easy using our MobilePay Invoice Api's.<br />
This document explains how to make a technical integration to the MobilePay Invoice product. The audience for this document is either technical integrators acting on behalf of merchants, or merchant creating their own integrations.


## <a name="integration"/> Integration     
Integrating to MobilePay invoice is technically a multistep process involving creating an application interacting with our systems via our API gateway, subscribing to the invoice product and calling the invoice RESTful API's.<br />

In the sections below, the following steps will be explained.
1. Merchant onboarding  
2. Subscribing to the Invoice product through the API gateway
3. Using OpenID Connect enabling integrators to call the invoice API on-behalf of merchants
4. How does the Invoice API look like and how is it called.

### <a name="merchant_onboarding"/> Merchant Onboarding
As a merchant wanting to send invoices via MobilePay, you enroll to the product via [MobilePay.dk](http://www.MobilePay.dk) or the MobilePay Business Administration portal. Here you will find information about our products as well as an easy way of enrolling. As an integrator or 3rd party service provider acting on behalf of a merchant, you need your customer to enroll to the product prior to doing the integration.<br />
During the first product enrollment, you, as a merchant, will be requested details about your company such as company size, type etc. You will also be asked to confirm the account(s) you wish to use. These informations are required in order for us to verify your company information to be able to transfer the money to you securely. <br />

During the enrollment you will also be given the option to define parameters such as name, logo, address, etc. for how your MobilePay Invoice Issuer should reflect your company in MobilePay.


<a name="apigateway"/>  

### API Gateway     

The first step needed in order to integrate to MobilePay invoice is obtaining access to the sandbox environment. The sandbox environment is a production like environment used in MobilePay to test technical integrations. The environment is located [here](https://sandbox-developer.mobilepay.dk/).<br />

Make sure that it is clear that the invoice product is wanted when requesting access. Once logged in, create an app representing your application. This will yield a client id and a secret -remember these as they are needed in every call. After this, navigate to the invoice product and subscribe with your newly created app. All facilities and information for testing connection will be available inside the sandbox environment when logged in.

<a name="openidconnect"/>  

### OpenID Connect   

When the merchant is onboarded, he has a user in MobilePay that is able to manage which products the merchant wishes to use. Not all merchants have the technical capabilities to make integrations to MobilePay, instead they may need to go through applications whith these capabilities. In order for this to work, the merchant must grant consent to an application with these capabilities. This consent is granted through mechanism in the [OpenID Connect](http://openid.net/connect/) protocol suite.<br />

The OpenID Connect protocol is a simple identity layer on top of the OAuth 2.0 protocol. Integrators are the same as clients in the OAuth 2.0 protocol. The first thing that must be done as a client is to go and register [here]( https://www.mobilepay.dk/da-dk/Erhverv/Pages/MobilePay-integrator.aspx). Once this is done the client must initiate the [hybrid flow](http://openid.net/specs/openid-connect-core-1_0.html#HybridFlowAuth) specified in OpenID connect. For invoices the client must request consent from the merchant using the 'invoice' scope. The authorization server in sandbox is located  https://api.sandbox.mobilepay.dk/merchant-authentication-openidconnect <br />

If the merchant grants consent, an authorization code is returned which the client must exchange for an id token, an access token and a refresh token. The refresh token is used to refresh ended sessions without asking for merchant consent again. This means that if the client receives an answer from the api gateway saying that the access token is invalid, the refresh token is exchanged for a new access token and refresh token.


An example of how to use OpenID connect in C# can be found [here](https://github.com/MobilePayDev/MobilePay-Invoice/tree/master/ClientExamples).

### <a name="openid-flow"></a>OpenID flow
![](assets/images/Diagram.png)

<a name="invoiceapi"/>      

## Invoice API

When the **Consent** between **Merchant** and the **Integrator** is established, use the merchant api endpoint to en-queue **Invoice Requests**. The invoice API consists of the following endpoints: <br />
1. Request Invoice `POST api/v1/merchants/{merchantId}/invoices`    
2. Get Invoice status `GET api/v1/merchants/{merchantId}/invoices/{invoiceId}/status`
3. Get Invoice issuers `GET api/v1/merchants/{merchantId}/invoiceissuers`
4. Get merchantId `GET api/v1/merchants/me`
5. Request Invoice Link `POST api/v1/merchants/{merchantId}/invoices/link`

### Invoice POST request
This endpoint accepts a JSON array of individual **Invoice Requests** to be processed asynchronously.

#### Request parameters

|Parameter             |Sub Parameter |Type        |Required  |Description                                       |Valid values|
|----------------------|--------------|------------|----------|--------------------------------------------------|------------|
|**InvoiceIssuer**  |              |guid        | required |*The ID of the invoicing department/branch of the merchant*|5e1210f9-4153-4fc3-83ec-2a8fc4843ea6|
|**ConsumerAlias**     |              |            | required |*Mobile alias of the MobilePay user to be invoiced*||
|    | **Alias**  |string      | required |*Alias value of the MobilePay user*|e.g. 004512345678, 12345678, +4512345678|
|    | **AliasType**  |string      | required |*Alias type of the MobilePay user, allowed values are: Phone number*|Phone|
|**ConsumerName**      |              |string      | required |*Full name of the MobilePay user*|Free text, Contact Name|
|**TotalAmount**       |              |number(0.00)| required |*The requested amount to be paid.*|>= 0.00, decimals separated with a dot.|
|**TotalVatAmount**    |              |number(0.00)| required |*VAT amount*|>= 0.00, decimals separated with a dot.|
|**CountryCode**       |              |string(2)   | required |*Country code*| DK |
|**CurrencyCode**      |              |string(3)   | required |*Currency code*|DKK |
|**ConsumerAddressLines**|            |string      | required |*Address of consumer receiving the invoice*|Free text|
|**DeliveryAddressLines**|            |string      |          |*Delivery address*|Free text|
|**InvoiceNumber**     |              |string      | required |*Invoice Number*|Free text e.g. 123456798ABCD|
|**IssueDate**         |              |date        | required |*Issue date of invoice*|ISO date format: YYYY-MM-DD|
|**DueDate**           |              |date        | required |*Payment due date. Must be between today and +400 days ahead, otherwise the Request will be declined.*|ISO date format: YYYY-MM-DD|
|**OrderDate**         |              |date        | required |*Order date of invoice*|ISO date format: YYYY-MM-DD|
|**Comment**           |              |string      |          |*Free text of additional information to the consumer*|Free text|
|**MerchantContactName**|             |string      |          |*Contact name for the individual who issued the invoice*|Free text, Name|
|**MerchantOrderNumber**|             |string      |          |*The ordernumber for the invoice used internally by the merchant*|Free text e.g. 123456798ABCD|
|**BuyerOrderNumber**|              |string      |          |*The ordernumber for the invoice used externally by the merchant*|Free text e.g. 123456798ABCD|
|**PaymentReference**  |              |string(60)  |          |*Reference used on the payment to do reconsilitaion. If not filled, invoice number will be used as reference*|Free text e.g. 123456798ABCD|
|**InvoiceLineItem[]** |              |            | required |*At least one invoice line is required*||
|    |**ArticleNumber**               |string      |          |*Article Number*|e.g. 123456ABC|
|    |**ArticleDescription**          |string      |          |*Article Descrition*|Free text|
|    |**VATRate**                     |number(0.00)|          |*VAT Rate of article*|>= 0.00, decimals separated with a dot.|
|    |**TotalVATAmount**              |number(0.00)|          |*Total VAT amount of article*|>= 0.00, decimals separated with a dot.|
|    |**TotalPriceIncludingVat**      |number(0.00)|          |*Total price of article including VAT*|>= 0.00, decimals separated with a dot.|
|    |**Unit**                        |string      |          |*Unit*|e.g. Pcs, Coli|
|    |**Quantity**                    |number(0.00)|          |*Quantity of article*|>= 0.00, decimals separated with a dot.|
|    |**PricePerUnit**                |number(0.00)|          |*Price per unit*|>= 0.00, decimals separated with a dot.|
|    |**PriceReduction**              |number(0.00)|          |*Price reduction*|>= 0.00, decimals separated with a dot.|
|    |**PriceDiscount**               |number(0.00)|          |*Price discount*|>= 0.00, decimals separated with a dot.|
|    |**Bonus**                       |number(0.00)|          |*Quantity of article*|>= 0.00, decimals separated with a dot.|


##### HTTP 202 Response body example
InvoiceId: 5e1210f9-4153-4fc3-83ec-2a8fc4843ea6

### Invoice status GET request
This endpoint is used to request the status of individual invoices

#### Request parameters

There is no JSON input model in this endpoint, instead, format the request the in the following way: <br />
**{merchantid:guid}/invoices/{invoiceid:guid}/status**

##### HTTP 200 Response body example

|Parameter Name     |Type     |Description               |Valid values                         |
|-------------------|---------|--------------------------|-------------------------------------|
|**InvoiceId**      | guid    |*The ID of the invoice*   |5e1210f9-4153-4fc3-83ec-2a8fc4843ea6 |
|**Status**         | string  |*Status of the invoice*   | 1: Created <br /> 2: Paid <br /> 3: Rejected <br /> 4: Expired |

### MerchantId GET request
This endpoint is used to get the merchant id associated with a merchant.

#### Request parameters

There is no JSON input model in this endpoint, instead, format the request the in the following way: <br />
`/api/v1/merchants/me`

##### HTTP 200 Response body example
MerchantId: 5e1210f9-4153-4fc3-83ec-2a8fc4843ea6

### InvoiceIssuers GET request
This endpoint is used to get the invoice issuers associated with a merchant.

#### Request parameters

There is no JSON input model in this endpoint, instead, format the request the in the following way: <br />
`/api/v1/merchants/{merchantid}/invoiceissuers`

##### HTTP 200 Response body example

|Parameter Name                   |Type         |Description           |Value                        |
|---------------------------------|-------------|----------------------|-----------------------------|
|**InvoiceIssuers**           | List        |*List of invoice issuers for a merchant* | {<br />"Id": "1f8288d9-4511-43ef-a1ce-667835470577", <br /> "Name": "Test Fik Issuer" <br /> "AccountType": "FIK" <br /> },<br /> { <br /> "Id": "3d579d95-5cbe-4e45-b3e0-3b73d37e8b9c", <br /> "Name": "TestName" <br /> "AccountType": "Account" <br /> } |


<a name="invoice-link"/>   

## Invoice Link

Merchant's can create an Invoice that can be paid by any MobilePay user. Merchant's back-end system must call the `POST /api/v1/merchants/{merchantid:guid}/invoices/link` endpoint in order to generate a **Link** refering to **Invoice**, which can be activated by the MobilePay user through the app or web browser.

#### Invoice Link POST request

This endpoint accepts a JSON object of Invoice Request to be processed asynchronously.

```json
{
  "InvoiceIssuer": "efd08c19-24cf-4833-a4a4-bfa7bd58fbb2",
  "ConsumerAlias": {
    "Alias": "+4577007700",
    "AliasType": "Phone"
  },
  "ConsumerName": "Consumer Name",
  "TotalAmount": "360",
  "TotalVATAmount": "72",
  "CountryCode": "DK",
  "CurrencyCode": "DKK",
  "ConsumerAddressLines": [
    "Paradisæblevej 13 1234 Andeby"
  ],
  "DeliveryAddressLines": [
    "Østerbrogade 120"
  ],
  "InvoiceNumber": "301",
  "IssueDate": "2018-02-12",
  "DueDate": "2018-03-12",
  "OrderDate": "2018-02-05",
  "DeliveryDate": "2018-02-10",
  "Comment": "Any comment",
  "MerchantContactName": "Snowboard gear shop",
  "MerchantOrderNumber": "938",
  "BuyerOrderNumber": "631",
  "PaymentReference": "186",
  "InvoiceArticles": [
    {
      "ArticleNumber": "1-123",
      "ArticleDescription": "Process Flying V Snowboard",
      "VATRate": 25,
      "TotalVATAmount": 72,
      "TotalPriceIncludingVat": 360,
      "Unit": "1",
      "Quantity": 1,
      "PricePerUnit": 288,
      "PriceReduction": 0,
      "PriceDiscount": 0,
      "Bonus": 5
    }      
  ]
}
```
#### <a name="InvoiceLink_paramters"></a>Request parameters

|Parameter             |Sub Parameter   |Type        |Required  |Description                                                 |Valid values|
|:---------------------|:---------------|:-----------|:---------|:-----------------------------------------------------------|:-----------|
|**InvoiceIssuer**     |                     | guid       | required |*The ID of the invoicing department/branch of the merchant*|5e1210f9-4153-4fc3-83ec-2a8fc4843ea6|
|**ConsumerAlias**     |                     | object     |          |*Mobile alias of the MobilePay user to be invoiced*| |
|                      | **Alias**           |string      |          |*Alias value of the MobilePay user*| e.g. 004512345678, 12345678, +4512345678|
|                      | **AliasType**       |string      |          |*Alias type of the MobilePay user, allowed values are: Phone number*| Phone |
|**ConsumerName**       |   | string | required  |*Full name of the MobilePay user*| Free text e.g. Contact Name|
|**TotalAmount**        |   | decimal | required |*The requested amount to be paid.*|>= 0.00, decimals separated with a dot.|
|**TotalVATAmount**     |   | decimal | required |*VAT amount*| >= 0.00, decimals separated with a dot. |
|**CountryCode**        |   | string | required |*Country code*| DK, FI |
|**CurrencyCode**       |   | string | required |*Currency code*| DKK, EUR |
|**ConsumerAddressLines**    |   | string[] | At least one |*Address of consumer receiving the invoice*| Free text |
|**DeliveryAddressLines**    |   | string[] |  |*Delivery address*| Free text |
|**InvoiceNumber**      |   | string | required |*Invoice Number*| Free text e.g. 123456798ABCD |
|**IssueDate**          |   | Date | required |*Issue date of invoice*| ISO date format: YYYY-MM-DD |
|**DueDate**            |   | Date | required |*Payment due date. Must be between today and +400 days ahead, otherwise the Request will be declined*| ISO date format: YYYY-MM-DD |
|**OrderDate**          |   | Date | required | | ISO date format: YYYY-MM-DD |
|**DeliveryDate**       |   | Date |  | |  |
|**Comment**            |   | string |  |*Free text of additional information to the consumer*| Free text |
|**MerchantContactName**     |   | string |  |*Contact name for the individual who issued the invoice*| Free text, Name |
|**MerchantOrderNumber**     |   | string |  |*The order number for the invoice used internally by the merchant*| Free text e.g. 123456798ABCD |
|**BuyerOrderNumber**        |   | string |  |*he ordernumber for the invoice used externally by the merchant*| Free text e.g. 123456798ABCD |
|**PaymentReference**        |   | string |  |*Reference used on the payment to do reconsilitaion. If not filled, invoice number will be used as reference*| Free text e.g. 123456798ABCD |
|**InvoiceArticles**         |   | list | required |*At least one invoice line is required*|  |
|               |  **ArticleNumber** | string |  |*Article Number*| e.g. 123456ABC |
|               | **ArticleDescription**  | string |  |*Article Descrition*| Free text |
|               | **VATRate** | decimal |  |*VAT Rate of article*| >= 0.00, decimals separated with a dot. |
|               | **TotalVATAmount** | decimal |  |*Total VAT amount of article*| >= 0.00, decimals separated with a dot. |
|               | **TotalPriceIncludingVat**  | decimal |  |*Total price of article including VAT*| >= 0.00, decimals separated with a dot. |
|               | **Unit**  | decimal |  |*Unit*| e.g. Pcs, Coli |
|               | **Quantity**  | decimal |  |*Quantity of article*| >= 0.00, decimals separated with a dot. |
|               | **PricePerUnit**  | decimal |  |*Price per unit*| >= 0.00, decimals separated with a dot. |
|               | **PriceReduction**  | decimal |  |*Price reduction*| >= 0.00, decimals separated with a dot. |
|               | **PriceDiscount**  | decimal |  |*Price discount*| >= 0.00, decimals separated with a dot. |
|               |  **Bonus**    | decimal |  |*Quantity of article*| >= 0.00, decimals separated with a dot. |


The response of `POST /api/v1/merchants/{merchantId}/invoices/link` contains two values: a unique id of the Invoice and a Link rel = user-redirect

##### HTTP 202 Response body example
```json
{
    "InvoiceId": "c5d4fde3-81e2-49de-8cfe-8c96f449e367",
    "Links": [
        {
            "Rel": "user-redirect",
            "Href":"https://api.sandbox.mobilepay.dk/invoice-restapi/api/v1/consumers/me/invoices/invoices/c5d4fde3-81e2-49de-8cfe-8c96f449e367/link"
        }
    ]
}
```
* The InvoiceId value can be used on the merchant’s back-end system to map a Invoice with a specific user on the merchant’s side.
* The link rel = user-redirect value contains the hyperlink reference address, which is structured in the following way: _https://&lt;mobile-pay-invoice-restapi&gt;/&lt;path-to-invoices&gt;/{invoice_id}/link_. The invoice_id property is of type guid and uniquely identifies the Invoice for the app to get the details and subsequently do an accept request.

The **Invoice Link** can be used in two ways:

1. Redirect the user automatically using the HTTP response **302** or **303**. Once the user is redirected, the MobilePay app will be opened to activate the **Invoice**.
2. E-mail the generated link to the user. Once the user clicks on the **Invoice Link**, the MobilePay app will be opened to activate the **Invoice**. Note, that the Invoice link will be valid only until the user accepts the Invoice or it will expire 30 days after due date.