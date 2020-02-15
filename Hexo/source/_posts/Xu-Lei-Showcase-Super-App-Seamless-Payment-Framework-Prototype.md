---
title: Xu Lei Showcase - Seamless Super App Prototype
tags:
  - showcase
  - super app
  - QR code
  - security
  - identity
  - java
  - android
  - payment
date: 2018-04-15 00:00:00
---
(*`Declare: the Seamless Super App Prototype is my personal project, and it does not have any relationship with PayPal. I will remove the PayPal logo very soon.`*)
Note: The size of the demo video is about 100MB, so it could be better to watch it in the `WiFi` network environment. The demo video is tested against the Chrome, Firefox and Safari browers in Mac. You can stop to watch it by operating the video control panel. Demo steps: 1. Scan McDonald’s QR code for seamless payment, 2. Scan Pizza Hut QR code for seamless payment.
<table><tr><th bgcolor="grey"><center>Scan to Pay for McDonald's</center></th><th bgcolor="grey"><center>Scan to Pay for Pizza Hut</center></th></tr><tr><td height="510"><video src="/videos/superapp/PayPalQR_McD.mov" width="100%" height="100%" poster="/images/superapp/poster.png" autoplay="autoplay" controls="controls" >Your browser or system may not support the media. Please use latest version of Chrome, Firefox or Safari in Mac.</video></td><td height="510"><video src="/videos/superapp/PayPalQR_PizH.mov" width="100%" height="100%" poster="/images/superapp/poster.png" autoplay="autoplay" controls="controls" >Your browser or system may not support the media. Please use latest version of Chrome, Firefox or Safari in Mac.</video></td></tr></table>

##Concept
`Super App`
Super App means opening for Integration, Integration and Integration!!!
(Refer to WeChat Little Program concept) The future apps allow users `Use and Leave` without downloading. 
`Seamless Authentication User Experience`
Seamless Login for the Authenticated User. For example, the users logged in their mobile app, they can use the system anywhere such as mobile, web app at laptop, and so forth.
Blur the lines of payment between online and offline.
Blur the lines between different systems.

##Design
![logic view](/images/superapp/logic_view.png "logic view")
![physical view](/images/superapp/physical_view.png "physical view")

##Security
<table><tr><th width="25%" bgcolor="grey"><center>Security Concern</center></th><th bgcolor="grey"><center>Security Challenge</center></th><th bgcolor="grey"><center>Solution</center></th></tr><tr><td bgcolor=#eeeeee> QR Code </td><td> The QR code is able to contain any information, valid or invalid. It will be risky to redirect user to the invalid site. </td><td> `Tokenized QR Code` signed by the payment gateway (here is JWT, Json Web Token). `HMAC` digital signature within the token ensures token can be verified by issuer only, and no one is able to change the token information as only the payment gateway hold the HMAC signature key. </td></tr><tr><td bgcolor=#eeeeee> Payment Process </td><td> Merchants should not play the role to conduct the payment process. For example, user buy $10 product, merchant might internally ask the payment gateway to charge the user $100 intently or by mistake. </td><td> Merchant site `natively direct the payment process to the payment gateway mobile app to allow users confirm the payment within the payment app`. (Android prototype tech challenge: child thread call redirect to main thread). User is able to make a final verification of their payment detail before clicking the pay button. Payment gateway will fully control the payment process securely. </td></tr><tr><td bgcolor=#eeeeee> Payment Status Notification </td><td> For the merchant perspective, it’s not secure to allow any client to notify and change the payment status. </td><td> By design, the merchant portal just allows payment gateway as the payment authority to notify the payment result. Merchant will generate the receipt based on the payment status for the specific order. </td></tr></table>

##Extension
a. Seamless authentication. I am an authenticated user at the mobile app. I should be able to seamlessly get authenticated at my iPad app, Desktop Web System wherever I am required to be authenticated. SSO, single sign on, to be the legal entity in the system cross platforms, QR code can bridge the gap.
b. Seamless authorization with payment. Merchants generate the bill online and users just need to open the payment app, Scan the QR code on the web page and Pay.  Blur the lines between the payment gateway and the merchant site. Username and Password are old days for authorization and authentication. Inconvenient and high risk to enter the credential through merchant site due to phishing sites.
c. Order ahead. For those used merchant tokens, they can be stored as the history or shortcut for the next time order ahead capability. 
d. Default integration with verified government sites. User can easily pay for their tax, water bill, phone bill, and so forth. Scenario payment to change user’s daily lives, and increase DAU (Daily Active User).
e. Integration with loyalty systems, restaurant bookings, advance paying for carpark to reserve it before you reaching the place, paying for season parking, etc.
