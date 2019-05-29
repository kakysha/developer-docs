---
description: >-
  You can store arbitrary data (any sequence of bytes) in Obyte public DAG. Some special types of data also supported, like 'text', 'profile', 'poll', etc.
---

# Sending data to DAG database

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