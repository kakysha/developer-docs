---
description: >-
  If you run an exchange, you will likely want to interact with your Obyte node
  via RPC interface.
---

# Running RPC service

By default, RPC service is not enabled for security reasons. To enable it, you should start your headless node differently: instead of `node start.js`, cd to `tools` folder and start RPC-enabled node by running [rpc\_service.js](https://github.com/byteball/headless-obyte/blob/master/tools/rpc_service.js):

```javascript
cd tools
node rpc_service.js
```

The node works as usual, plus it listens on port 6332 of loop-back interface \(configured in [conf.js](https://github.com/byteball/headless-byteball/blob/master/conf.js) or conf.json\) for JSON-RPC commands. The commands are `getnewaddress`, `getbalance`, `listtransactions`, `sendtoaddress`.

## getinfo

The command returns information about the current state of the DAG.

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"getinfo", "params":{} }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":{"last_mci":253151,"last_stable_mci":253120,"count_unhandled":0},"id":1}
```

The command has no parameters and returns and object with 3 keys:

* `last_mci`: the highest known main chain index \(MCI\)
* `last_stable_mci`: last stable MCI
* `count_unhandled`: number of unhandled units in the queue. Large number indicates that sync is still in progress, 0 or small number means that the node is synced \(it can occasionally go above 0 when new units are received out of order\).

## getnewaddress

This command generates a new address in your wallet. You will likely want to use it to create a new deposit address and bind it to a user account.

Example usage:

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"getnewaddress", "params":{} }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":"QZEM3UWTG5MPKYZYRMUZLNLX5AL437O3","id":1}
```

The command has no parameters and the response is a newly generated wallet address \(32-character string\).

## validateaddress

This command validates a wallet address and returns `true` or `false`.

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"validateaddress", "params":["QZEM3UWTG5MPKYZYRMUZLNLX5AL437O3"] }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":true,"id":1}
```

You will likely want to use it before saving a withdrawal address for a customer.

## getbalance

Returns the balance of the specified address or the entire wallet.

Example usage for querying wallet balance:

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"getbalance", "params":{} }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":{"base":{"stable":8000,"pending":0}},"id":1}
```

Querying balance of an individual address:

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"getbalance", "params":["QZEM3UWTG5MPKYZYRMUZLNLX5AL437O3"] }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":{"base":{"stable":5000,"pending":0}},"id":1}
```

To query the balance of the entire wallet, parameters must be empty. To query the balance of an individual address, pass it as the only element of the params array.

The response is an object, keyed by asset ID \("base" for Bytes\). For each asset, there is another nested object with keys `stable` and `pending` for stable and pending balances respectively. Balances are in the smallest units \(bytes for the native currency\), they are always integers.

If the queried address is invalid, you receive error "invalid address". If the address does not belong to your wallet, you receive error "address not found".

## listtransactions

Use it to get the list of transactions on the wallet or on a specific address.

Example request for transactions on the entire wallet \(all addresses\):

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"listtransactions", "params":{"since_mci": 1234} }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":[
  {"action":"received","amount":3000,"my_address":"YA3RYZ6FEUG3YEIDIJICGVPD6PPCTIZK","arrPayerAddresses":["EENED5HS2Y7IJ5HACSH4GHSCFRBLA6CN"],"confirmations":0,"unit":"sALugOU8fjVyUvtfKPP0pxlE74GlPqOJxMbwxA1B+eE=","fee":588,"time":"1490452729","level":253518},
  {"action":"received","amount":5000,"my_address":"QZEM3UWTG5MPKYZYRMUZLNLX5AL437O3","arrPayerAddresses":["UOOHQW4ZKPTII4ZEE4ENAM5PC6LWAQHQ"],"confirmations":1,"unit":"vlt1vzMtLCIpb8K+IrvqdpNLA9DkkNAGABJ420NvOBs=","fee":541,"time":"1490452322","level":253483}
],"id":1}
```

On individual address:

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"listtransactions", "params":["QZEM3UWTG5MPKYZYRMUZLNLX5AL437O3"] }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":[
  {"action":"received","amount":5000,"my_address":"QZEM3UWTG5MPKYZYRMUZLNLX5AL437O3","arrPayerAddresses":["UOOHQW4ZKPTII4ZEE4ENAM5PC6LWAQHQ"],"confirmations":0,"unit":"vlt1vzMtLCIpb8K+IrvqdpNLA9DkkNAGABJ420NvOBs=","fee":541,"time":"1490452322","level":253483}
],"id":1}
```

To query the transactions on an individual address, pass it as the only element of the `params` _array_. In this case, only transactions in _bytes_ are returned. If the passed address is invalid, you receive error "invalid address".

In all other cases, `params` is an _object_.

To get the list of transactions in a particular asset, set an `asset` parameter in `params`. If `asset` is `null` or omitted, transactions in bytes will be returned.

To query the list of transactions since a particular main chain index \(MCI\), specify `since_mci`property in the `params` object, e.g. `"params": {"since_mci":254000}` or `"params": {"since_mci":254000; "asset": "f2TMkqij/E3qx3ALfVBA8q5ve5xAwimUm92UrEribIE="}`. The full list of matching transactions will be returned, however large it is.

To query an individual transaction, specify its unit in the `params` object: `"params": {"unit":"vlt1vzMtLCIpb8K+IrvqdpNLA9DkkNAGABJ420NvOBs="}`.

The response is an array of transactions in reverse chronological order.

Each transaction is described by an object with the following fields:

* `action`: string, one of `invalid`, `received`, `sent`, `moved`.
* `amount`: integer, amount of the transaction in the smallest units
* `my_address`: string, the address that belongs to your wallet and received funds \(for `received`and `moved` only\)
* `arrPayerAddresses`: array of payer addresses \(for `received` only\)
* `confirmations`: integer 0 \(pending\) or 1 \(final\), shows confirmation status of the transaction
* `unit`: string, unit of the transaction \(also known as transaction id\)
* `fee`: integer, fee in bytes
* `time`: integer, seconds since the Epoch
* `level`: integer, level of the unit in the DAG
* `mci`: integer, MCI of the unit. It can change while the unit is still pending and becomes immutable after the unit gets final

#### Waiting for deposits

To operate an exchange, you'll want to wait for new deposits using the following scenario:

* call `getinfo` and remember `last_stable_mci`, you'll use it in the following \(not this one\) iteration;
* call `listtransactions` with `since_mci` set to `last_stable_mci` remembered from the previous \(not this one!\) iteration;
* look for transactions with `action`s `received` and `moved` \(you need `moved` in case one user withdraws to a deposit address of another user\) and identify the user by `my_address`;
* wait;
* repeat the cycle.

In each cycle, save `last_stable_mci` in persistent storage and start from it after the wallet is restarted.

If you support several Obyte-issued assets, you'll need to call `listtransactions` for each asset individually.

## sendtoaddress

Use this command to withdraw bytes or another asset. Example usage:

```bash
$ curl --data '{"jsonrpc":"2.0", "id":1, "method":"sendtoaddress", "params":["BVVJ2K7ENPZZ3VYZFWQWK7ISPCATFIW3", 1000] }' http://127.0.0.1:6332
{"jsonrpc":"2.0","result":"vuudtbL5ASwr0LJZ9tuV4S0j/lIsotJCKifphvGATmU=","id":1}
```

There are 3 parameters to this command:

* destination address \(32-character string\);
* amount in bytes or smallest indivisible units \(integer\);
* \(optional\) asset \(44-character string or `null`\). If missing or `null`, the asset is assumed to be bytes.

On success, the response is the unit of the spending transaction \(string\). If the command failed, an error message is returned. It is possible that the command returns error due to lack of confirmed funds, in this case you should retry the command in a few minutes. If the request timed out without returning an error, do not retry automatically, the funds might be already sent!

If the destination address is invalid, the command returns error "invalid address". To avoid this, it is recommended to validate user-entered withdrawal address using `validateaddress` above or [ocore](https://github.com/byteball/byteballcore) library:

```javascript
var validationUtils = require("ocore/validation_utils.js");
if (!validationUtils.isValidAddress(address)){
  // notify user that the entered wallet address is invalid
}
```

