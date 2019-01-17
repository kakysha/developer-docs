---
description: >-
  Chatbots communicate with users and offer various services. These services are
  usually connected with payments.
---

# Writing chatbots on Obyte

## Prerequisites

To run a chatbot, you need to [setup a headless node](setting-up-headless-wallet.md) first. Headless nodes are full nodes by default. A subset of the functionality might work in light nodes too, but keep in mind that it was designed for full nodes only.

Create a new node.js package for your bot. You will definitely need modules from `ocore`\(add it to your project's dependencies\) and if you are going to send payments, you will also need `headless-obyte`. Your `package.json` should list these dependencies:

{% code-tabs %}
{% code-tabs-item title="package.json" %}
```javascript
"dependencies": {
	"headless-obyte": "git+https://github.com/byteball/headless-obyte.git",
	"ocore": "^0.2.33",
	.....
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

In your module, `require()` wallet.js even if you are not using any of its functions \(it handles messages from the hub\):

```javascript
require('ocore/wallet.js');
```

### Configuration

In your configuration file \([conf.js in project folder or conf.json in user folder](https://github.com/byteball/byteballcore#configuring)\) specify your bot name and a pairing secret that will be part of your pairing code:

{% code-tabs %}
{% code-tabs-item title="conf.js" %}
```javascript
exports.deviceName = 'My test bot';
exports.permanent_pairing_secret = '0000';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

When you start your node, it will print its full pairing code:

```text
====== my pairing code: A2WMb6JEIrMhxVk+I0gIIW1vmM3ToKoLkNF8TqUV5UvX@byteball.org/bb#0000
```

The pairing code consists of your node's public key \(device public key\), hub address, and pairing secret \(after \#\).

Publish this pairing code on your website as a link with `byteball:` scheme, users will be able to open a chat with your bot by clicking your link \(the link opens in their Obyte app and starts a chat\):

```markup
<a href="byteball:A2WMb6JEIrMhxVk+I0gIIW1vmM3ToKoLkNF8TqUV5UvX@byteball.org/bb#0000">Chat with my test bot</a>
```

You also need this pairing code to add your bot to the [Bot Store](https://medium.com/byteball/byteball-bot-store-has-launched-c546e9e38ab5).

### Email notifications

Most bots out there expect user's machine to have UNIX sendmail and by default `sendmail` function in `mail` module will try to use that, but it is possible to configure your node to use SMTP relay. This way you could use Gmail or Sendmail SMTP server or even something like Mailtrap.io \(excellent for testing purposes if you don't want the actual email recipient to receive your test messages\). This is how the configuration would look:

{% code-tabs %}
{% code-tabs-item title="conf.json" %}
```javascript
{
	"smtpTransport": "relay",
	"smtpRelay": "smtp.mailtrap.io",
	"smtpUser": "MAILTRAP_INBOX_USER",
	"smtpPassword": "MAILTRAP_INBOX_PASSWORD",
	"smtpSsl": false,
	"smtpPort": 2525,
	"admin_email": "admin@example.com",
	"from_email": "bot@example.com"
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### SQL Database

If you need to inspect the DAG \(most likely, you need\) and see the details of any transaction, require the `db` module

```javascript
var db = require('ocore/db.js');
```

and run any SQL queries on your copy of the Obyte database \(sqlite file is located in [same folder as configuration](https://github.com/byteball/byteballcore#configuring)\). You can also add your custom tables. Obviously, you don't want to modify any data outside your custom tables.

Obyte nodes will use `sqlite` database by default because it needs zero configuration, but it is possible to use MySQL or MariaDB too with a configuration like this:

{% code-tabs %}
{% code-tabs-item title="conf.json" %}
```javascript
{
	"storage": "mysql",
	"database": {
		"host"     : "localhost",
		"user"     : "DATABASE_USER",
		"password" : "DATABASE_PASSWORD",
		"name"     : "DATABASE_NAME"
	}
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Receiving pairing notifications

When a user pairs his device with your bot \(e.g. by clicking the link to your bot\), you receive a pairing notification and can welcome the user:

```javascript
eventBus.on('paired', function(from_address, pairing_secret){
	var device = require('ocore/device.js');
	device.sendMessageToDevice(from_address, 'text', 'Hi! I am bot.');
});
```

The behavior of your event handler can depend on `pairing_secret`, the second argument of the event handler. For example, you can have a one-time \(non-permanent\) pairing secret equal to session ID on your website; after the user clicks the pairing link and opens chat, you can link his chat session with his web session, see [Authenticating users on websites](../tutorials-for-newcomers/log-in-on-website-with-byteball.md).

### Accept any pairing secret

If you wish to accept any pairing secret then you can do that by changing the pairing secret row in  configuration file into this:

{% code-tabs %}
{% code-tabs-item title="conf.js" %}
```javascript
exports.permanent_pairing_secret = '*';
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Receiving chat messages

To receive chat messages, subscribe to 'text' events on event bus:

```javascript
var eventBus = require('ocore/event_bus.js');
eventBus.on('text', function(from_address, user_message){
    // your code here
});
```

`from_address` is user's device address \(not to be confused with payment addresses\), `user_message` is their message.

## Sending chat messages

Messages to user's device can be sent with `sendMessageToDevice` function in `device` module:

```javascript
var device = require('ocore/device.js');
device.sendMessageToDevice(user_device_address, 'text', 'Message from bot to user');
```

So, in case we want to echo back what user wrote, we could combine those two above examples into this:

```javascript
var eventBus = require('ocore/event_bus.js');
eventBus.on('text', function(from_address, user_message){
    var device = require('ocore/device.js');
    device.sendMessageToDevice(from_address, 'text', user_message);
});
```

Great success, we now have a bot, which is like a parrot \(repeating everything you wrote to it\).

## Predefined chat commands

To give access to predefined commands, format your responses this way:

```text
click this link: [Command name](command:command code)
```

The user will see the text in square brackets "Command name", it will be highlighted as a link, and when the user clicks it, his app will send `command code` text to your bot.

### Command suggestion

Sometimes you might want to suggest the command without actually sending it immediately, so user could have the possibility to edit the command before sending, this could be done like this:

```text
Example command: we suggest to [buy (number) apples](suggest-command:buy 5 apples)
```

## Payments

One of the core features that almost every existing Obyte chatbot has is sending and receiving payments. This enables bot developers immediately add payment to the service they could be providing without the need for any other third-party payment processor.

### Requesting payments

When you include a valid Obyte address anywhere in the text of your response to the user, the address will be automatically highlighted in the user's chat window, and after clicking it the user will be able to pay arbitrary amount of arbitrary asset to this address.

When you want to request a specific amount of a specific asset, format your payment request this way:

```text
Any text before [payment description, will be ignored](byteball:YOUR_WALLET_ADDRESS?amount=123000&asset=base) any text after
```

Amount is in the smallest units, such as bytes. If you omit `&asset=...` part, base asset \(bytes\) is assumed. If you want to request payment in another asset, indicate its identifier such as `oj8yEksX9Ubq7lLc+p6F2uyHUuynugeVq4+ikT67X6E=` for blackbytes \(don't forget to url-encode it\).

You will likely want to generate a unique payment address per user, per transaction. This code sample might help:

```javascript
var walletDefinedByKeys = require('ocore/wallet_defined_by_keys.js');
walletDefinedByKeys.issueNextAddress(wallet, 0, function(objAddress){
	var wallet_address = objAddress.address;
	// work with this address, then send it over to the user
	device.sendMessageToDevice(user_device_address, 'text', "Please pay to "+wallet_address);
});
```

### Waiting for payments

If you include a headless wallet

```javascript
var headlessWallet = require('headless-obyte');
```

you can get notified when any of your addresses receives a payment

```javascript
eventBus.on('new_my_transactions', function(arrUnits){
	// react to receipt of payment(s)
});
```

`arrUnits` is an array of units \(more accurately, unit hashes\) that contained any transaction involving your addresses. The event `new_my_transactions` is triggered for outgoing transactions too, you should check if the new transaction credits one of the addresses you are expecting payments to.

### Waiting for finality of payments

To get notified when any of your transactions become stable \(confirmed\), subscribe to `my_transactions_became_stable` event:

```javascript
eventBus.on('my_transactions_became_stable', function(arrUnits){
	// react to payment(s) becoming stable
});
```

`arrUnits` is again the array of units that just became stable and they contained at least one transaction involving your addresses.

The above events work in both full and light nodes. If your node is full, you can alternatively subscribe to event `mci_became_stable` which is emitted each time a new main chain index \(MCI\) becomes stable:

```javascript
eventBus.on('mci_became_stable', function(mci){
	// check if there are any units you are interested in 
	// that had this MCI and react to their becoming stable
});
```

### Sending payments

To send payments, you need to include a headless wallet

```javascript
var headlessWallet = require('headless-obyte');
```

and use this function:

```javascript
headlessWallet.issueChangeAddressAndSendPayment(asset, amount, user_wallet_address, user_device_address, function(err, unit){
	if (err){
		// something went wrong, maybe put this payment on a retry queue
		return;
	}
	// handle successful payment
});
```

`asset` is the asset you are paying in \(`null` for bytes\), `amount` is payment amount in the smallest units. If the payment was successful, you get its `unit` in the callback and can save it or watch further events on this unit.

There are many other functions for sending payments, for example sending multiple payments in multiple assets at the same time, see `exports` of [https://github.com/byteball/headless-obyte/blob/master/start.js](https://github.com/byteball/headless-byteball/blob/master/start.js).

## Sending data to DAG database

To send data to DAG database, code below is a minimal that you can use. In case when you want to send data or text from specific address, you will need to look into links to other code examples below.

```javascript
var headlessWallet = require('headless-obyte');
var eventBus = require('ocore/event_bus.js');
var objectHash = require('ocore/object_hash.js');

eventBus.on('headless_wallet_ready', () => {
	var json_data = {
		foo: 'bar'
	};
	var objMessage = {
		app: 'data',
		payload_location: 'inline',
		payload_hash: objectHash.getBase64Hash(json_data),
		payload: json_data
	};
	var opts = {
		messages: [objMessage]
	};

	​headlessWallet.sendMultiPayment(opts, function(err, unit){
		if (err){
			// something went wrong, maybe put this transaction on a retry queue
			return;
		}
		// handle successful payment
	});
});
```

Besides `data`, there are [other types of message apps](https://github.com/byteball/byteballcore/blob/master/writer.js) too, like `data_feed`, `attestation`, `profile` , `poll` and `text`. Code examples for these are available in ["/play" folder of headless-wallet](https://github.com/byteball/headless-byteball/tree/master/play).

If you wish to send data to DAG database periodically then full code examples for [price oracle](https://github.com/byteball/byteball-data-feed), [bitcoin oracle](https://github.com/byteball/btc-oracle) or [sports oracle](https://github.com/byteball/sports-oracle) can be found on Github.

## Offering smart contracts

When you want to create a new smart contract with a user, your sequence is usually as follows:

* you ask the user to send his payment address \(it will be included in the contract\)
* you define a new contract using the user's address and your address as parties of the contract
* you pay your share to the contract
* at the same time, you send a specially formatted payment request \(different from the payment request above\) to the user to request his share. You start waiting for the user's payment
* the user views the contract definition in his wallet and agrees to pay
* you receive the payment notification and wait for it to get confirmed
* after the payment is confirmed, the contract is fully funded

Most steps are already familiar, we'll discuss creating and offering contracts below. These two steps are similar to what happens in the GUI wallet when a user designs a new contract.

### Creating contract definition

Create a JSON object that defines the contract:

```javascript
var arrDefinition = ['or', [
	['and', [
		['address', USERADDRESS],
		conditions when user can unlock the contract
	]],
	['and', [
		['address', MYADDRESS],
		conditions when I can unlock the contract
	]]
]];
```

Create another object that describes the positions of your and user addresses in the above definition:

```javascript
var device = require('ocore/device.js');
var assocSignersByPath = {
	'r.0.0': {
		address: user_address,
		member_signing_path: 'r', // unused, should be always 'r'
		device_address: user_device_address
	},
	'r.1.0': {
		address: my_address,
		member_signing_path: 'r', // unused, should be always 'r'
		device_address: device.getMyDeviceAddress()
	}
};
```

The keys of this object are `r` \(from "root"\) followed by indexes into arrays in the definition's `or`and `and` conditions. Since the conditions can be nested, there can be many indexes, they are separated by dot.

Then you create the smart contract address:

```javascript
var walletDefinedByAddresses = require('ocore/wallet_defined_by_addresses.js');
walletDefinedByAddresses.createNewSharedAddress(arrDefinition, assocSignersByPath, {
	ifError: function(err){
		// handle error
	},
	ifOk: function(shared_address){
		// new shared address created
	}
});
```

If the address was successfully created, it was also already automatically sent to the user, so the user's wallet will know it.

More definition examples can be seen on [smart contracts definitions page](../smart-contracts.md).

### Sending a request for contract payment

To request the user to pay his share to the contract, create the below javascript object `objPaymentRequest` which contains both the payment request and the definition of the contract, encode the object in base64, and send it over to the user:

```javascript
var arrPayments = [{address: shared_address, amount: peer_amount, asset: peerAsset}];
var assocDefinitions = {};
assocDefinitions[shared_address] = {
	definition: arrDefinition,
	signers: assocSignersByPath
};
var objPaymentRequest = {payments: arrPayments, definitions: assocDefinitions};
var paymentJson = JSON.stringify(objPaymentRequest);
var paymentJsonBase64 = Buffer(paymentJson).toString('base64');
var paymentRequestCode = 'payment:'+paymentJsonBase64;
var paymentRequestText = '[your share of payment to the contract]('+paymentRequestCode+')';
device.sendMessageToDevice(correspondent.device_address, 'text', paymentRequestText);
```

The user's wallet will parse this message, display the definition of the contract in a user readable form, and offer the user to pay the requested amount. Your payment-waiting code will be called when the payment is seen on the DAG.

## Code samples

For working samples of code that implements the above functions, see the source code of existing chatbots:

* [Bot example](https://github.com/byteball/bot-example): start from here
* [Faucet](https://github.com/byteball/byteball-faucet): sending payments
* [ICO bot](https://github.com/byteball/ico-bot): sending and receiving payments
* [Merchant](https://github.com/byteball/byteball-merchant): receiving payments without storage of private keys
* [Flight delay insurance](https://github.com/byteball/flight-delays-insurance): offering contracts
* [Internal asset exchange](https://github.com/byteball/byteball-exchange): offering contracts
* [GUI wallet](https://github.com/byteball/byteball): not a bot, but you'll find code for offering contracts

