<a href="https://www.ambisafe.co/">![test](img/logo_red.png)</a>
**********
# Adding EToken based currency as a payment method
This guide provides a simple way to integrate any EToken cryptocurrency to your website and how our products can accelerate and facilitate this.

### Integration steps
Let's say that we want to work with CryptoCarbon currency. 

##### 1. Create payment address
First you need to register new account on the target cryptocurrency webwallet. In our case we are following to https://cryptocarbon.co.uk and register new account.
Our goal - receive valid address to receive payments.  
After registration steps you need to find your wallet address - 42 symbols hexadecimal string. It looks like this:  
![address](http://dl1.joxi.net/drive/2016/08/22/0015/1469/1037757/57/a39d45711c.png)

##### 2. Subscribe to notifications
Now you are almost ready to receive payments, but first you need to subscribe to events to know that payment has happened.  
Move to your Ambisafe account (sign up [here](https://www.ambisafe.co/user/register/) if you don’t have one) and click on "Notificator Service" in account menu and then "Add Filter". There are several templates and by default "EToken" is active, that is what we need.  
![empty filter form](http://dl1.joxi.net/drive/2016/08/30/0015/1469/1037757/57/8335e12a57.png)

First field is a **Name** of filter. You can create many filters, so adequate name can help you to distinguish one filter from another.

**Currency symbol**. Here we need to choose our desired currency CryptoCarbon (CC) from dropdown menu.

There are **prod** and **stage** EToken contracts, so you can create filters for different environments. Currently we want prod.

**Event name**. EToken has many events, we will look at them in the advanced tutorials, but currently we need only one - Transfer event.

**Confirmations** means how many blocks need to appear in the network before you will receive notification. If you want start receiving notifications immediately, you can leave this field empty - it has 1 block confirmation by default.

Use the **Callback URL** field to set URL endpoint from your website or other webservice. It will be used to send POST request with notification in JSON format.

**Filtered by** section holds additional rules for our filter to make it more specific.  
Rule is very straightforward. Every event has several fields and we can filter it by some values. Currently we need only one rule to receive all notifications. Transfer event has field **to**, that is, recipient. This is our wallet address that we have created before. So we just need to click on "+" button and add new rule with the value “to equal [YOUR_ADDRESS]” using dropdown menus.  
All rules are combined by AND operator, so be careful (more about rules you can read in the advanced tutorial).

Finally our filter will look like this:
![create filter form](http://dl2.joxi.net/drive/2016/08/30/0015/1469/1037757/57/2026a1bc46.png)

Click "Save" and be ready to receive notifications!

##### 3. Invoices and payments
To allow user to pay invoice, you need to create a link with a special custom protocol - EToken URI scheme. The purpose of this URI scheme is to enable users to easily make payments by simply clicking links on webpages or scanning QR Codes.  
It has such format:
```
web+[PROTOCOL_NAME]:[YOUR_ADDRESS]?amount=[AMOUNT]&reference=[INVOICE_ID]  
```

Here **PROTOCOL_NAME** is the name of protocol, which you can ask owner of currency. In our case, it is "cryptocarbon".  
**YOUR_ADDRESS** is your wallet address.  
**AMOUNT** is the price and **INVOICE_ID** is the unique ID of invoice at your choice, e.g. from database.  
For example, if user wants to buy iPhone for 42 CryptoCarbon, link with EToken URI will look like this:  
```html
<a href="web+cryptocarbon:0xfab8b376b3acc29b58b81296ffe7981164b764b9?amount=42&reference=AA-0123456789">Pay With CC</a>
```
When user clicks on link, he will be redirected to his webwallet with filled form to pay your invoice.

>Pay attention that user may not have this protocol installed at his browser. For this reason we recommend to show address, amount and invoice ID in text format as well. E.g. “Payment instructions: please send [AMOUNT] CC to address [YOUR_ADDRESS] and specify this invoice: [INVOICE_ID] as a comment to your transaction".

##### 4. Handle notifications
Notification Service will send notifications to your endpoints on each successful user payment according to your filters information.
This notifications will be HTTP POST request with json body like this (PHP example [here](https://github.com/Ambisafe/etoken-server-side-examples/blob/master/callback-handler.php)):
```json
{  
  "from":"0x1ff21eca1c3ba96ed53783ab9c92ffbf77862584",
  "eventName":"Transfer",
  "eventData":{  
    "reference":"[USER_INVOICE_ID]",
    "version":1,
    "symbol":"CC",
    "from":"0x1ff21eca1c3ba96ed53783ab9c92ffbf77862584",
    "to":"[YOUR_ADDRESS]",
    "value":3000
  },
  "transactionHash":"0xb8ad15e40cc01695ac996ddbecb3c1f5536dbd2c3d07d140387f71017ff8cb57",
  "timestamp":1465909825,
  "value":0,
  "blockNumber":1702807,
  "to":"[YOUR_ADDRESS]",
  "logIndex":2,
  "transactionIndex":0,
  "confirmations":1
}
```
>**NOTE:** value are presented in a base currency units, e.g. CryptoCarbon has base unit 6 means that 3000 tokens should be treated as 0.003.

After receiving notification you can verify it by checking signature.
Signature is a Base64 encoded SHA256 HMAC of HTTP body by your secret key. It is stored in Signature HTTP Header.

There are many fields, but currently we are interested in **eventData** section: to, reference and value fields.  
In **to** we can see our wallet address and in **reference** an invoice id generated for user. You need to check the **value** and if all the information is correct, mark this invoice as successful.  
Ta-da! You've received your first payment to your address and can finally send  the desired stuff to user!