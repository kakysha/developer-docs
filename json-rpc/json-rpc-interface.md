---
description: >-
  If you are developing a service on Byteball and your programming language is
  node.js, your best option is to just require() the byteball modules that you
  need.
---

# JSON RPC interface

For exchanges, there is a specialized [JSON RPC service](running-rpc-service.md). However it is limited and exposes only those functions that the exchanges need.

If you are developing a service on Byteball and your programming language is node.js, your best option is to just `require()` the byteball modules that you need \(most likely you need [headless-byteball](https://github.com/byteball/headless-byteball) and various modules inside [byteballcore](https://github.com/byteball/byteballcore)\). This way, you'll also be running a byteball node in-process.

If you are programming in another language, or you'd like to run your byteball node in a separate process, you can still access many of the functions of `headless-byteball` and `byteballcore`modules by creating a thin RPC wrapper around the required functions and then calling them via JSON-RPC.

To expose the required functions via JSON-RPC, create a small node.js module that depends on `headless-byteball`, `byteballcore` \(and any other byteball modules\) and [RPCify](https://github.com/byteball/rpcify):

```javascript
var rpcify = require('rpcify');
var eventBus = require('byteballcore/event_bus.js');

// this is a module whose methods you want to expose via RPC
var headlessWallet = require('headless-byteball'); 
var balances = require('byteballcore/balances.js'); // another such module

// most of these functions become available only after the passphrase is entered
eventBus.once('headless_wallet_ready', function(){

	// start listening on RPC port
	rpcify.listen(6333, '127.0.0.1');

	// expose some functions via RPC
	rpcify.expose(headlessWallet.issueChangeAddressAndSendPayment);
	rpcify.expose(balances.readBalance, true);
	rpcify.expose([
		headlessWallet.readFirstAddress,
		headlessWallet.readSingleWallet,
		headlessWallet.issueOrSelectAddressByIndex
	], true);

});
```

Start this module \(which also includes a byteball node\) and you'll be able to call the exposed functions via RPC from a program in any language. See the documentation about [RPCify](https://github.com/byteball/rpcify) for more details.

