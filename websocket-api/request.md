---
description: 'This is a message type, which gets a reply with a response type message.'
---

# Request

Example:

```javascript
const network = require('ocore/network.js');
const ws = network.getInboundDeviceWebSocket(device_address);

// function parameters: websocket, command, params, bReroutable, callback
var tag = network.sendRequest(ws, 'get_witnesses', null, false, getParentsAndLastBallAndWitnessListUnit);

function getParentsAndLastBallAndWitnessListUnit(ws, req, witnesses) {
  var params = {
    witnesses: witnesses
  };
  network.sendRequest(ws, 'light/get_parents_and_last_ball_and_witness_list_unit', params, false,
    function(ws, req, response) {
      console.log(response);
    }
  );
}

//you can have your own timeout logic and delete a pending request like this
setTimeout(function() {
  var deleted = network.deletePendingRequest(ws, tag); // true if request was deleted
}, 30*1000);
```

Following is a list of `request` type JSON messages that are sent over the network:

### **Get node list**

This requests returns response with nodes that accept incoming connections.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_peers'
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: [
          "wss://relay.papabyte.com/bb",
          "wss://hub.byteball.ee",
          "wss://byteball-hub.com/bb",
          ...
        ]
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get a list of witnesses**

This requests returns response with list of witnesses.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_witnesses'
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: [
          "4GDZSXHEFVFMHCUCSHZVXBVF5T2LJHMU",
          "BVVJ2K7ENPZZ3VYZFWQWK7ISPCATFIW3",
          "DJMMI5JYA5BWQYSXDPRZJVLW3UGL3GJS",
          ...
        ]
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get transaction data**

This requests returns response with unit data. Example shows a unit, which created a new asset.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_joint',
        params: '0xXOuaP5e3z38TF5ooNtDhmwNkh1i21rBWDvrrxKt0U='
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {
          "joint": {
            "unit": {
              "unit": "0xXOuaP5e3z38TF5ooNtDhmwNkh1i21rBWDvrrxKt0U=",
              "version": "1.0",
              "alt": "1",
              "witness_list_unit": "oj8yEksX9Ubq7lLc+p6F2uyHUuynugeVq4+ikT67X6E=",
              "last_ball_unit": "AV4qvQ5JuI7J3EO5pFD6UmMu+j0crPbH17aJfigozrc=",
              "last_ball": "Ogq4+uR6LaDspK89MmyydxRchgdN/BYwXvWXwNCMubQ=",
              "headers_commission": 344,
              "payload_commission": 607,
              "main_chain_index": 2336026,
              "timestamp": 1524844417,
              "parent_units": [
                "IFiHjYbzuAUC9TwcTrmpvezjTKBl6nIXjgjoKuVyCz0="
              ],
              "authors": [
                {
                  "address": "AM6GTUKENBYA54FYDAKX2VLENFZIMXWG",
                  "authentifiers": {
                    "r": "MBWlL31OkibUXhoxmwIcWB/fAx1uWdNTE8PYTBNeN3w+tev4N1anOXjGtXiGW4whW3PfJeTO9fA5WxwyqK/m2w=="
                  }
                }
              ],
              "messages": [
                {
                  "app": "data",
                  "payload_hash": "MiuMtTSgP0brPQijsRc0igb+dxcrkLjXWRE253AE/S8=",
                  "payload_location": "inline",
                  "payload": {
                    "asset": "IYzTSjJg4I3hvUaRXrihRm9+mSEShenPK8l8uKUOD3o=",
                    "decimals": 0,
                    "name": "WCG Point by Byteball",
                    "shortName": "WCG Point",
                    "issuer": "Byteball",
                    "ticker": "WCG",
                    "description": "WCG Point is a honorific token, a recognition of contributing to World Community Grid projects. The token is not transferable, therefore, it cannot be sold and the balance reflects a lifetime contribution to WCG. Some services might choose to offer a privilege to users with large balance of this token."
                  }
                },
                {
                  "app": "payment",
                  "payload_hash": "FaVxPdeRKTgcO/LCDtu/YUZego8xyrnpUMF3V/sujr8=",
                  "payload_location": "inline",
                  "payload": {
                    "inputs": [
                      {
                        "unit": "1p85M8VSRDaAkNoCAdtMrv6cVGpPNNeVJpLzOkzWLk8=",
                        "message_index": 1,
                        "output_index": 0
                      }
                    ],
                    "outputs": [
                      {
                        "address": "AM6GTUKENBYA54FYDAKX2VLENFZIMXWG",
                        "amount": 17890
                      }
                    ]
                  }
                }
              ]
            },
            "ball": "Ef36OjK5m6lQMII4S9MN3iGtQcsvTzzQFLKQZ44UBCg="
          }
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Send transaction data**

This requests returns response whether composed unit was accepted or not. Unit object can be composed with `ocore/composer.js`.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'post_joint',
        params: {Object}
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'accepted'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Send heartbeat that you don't sleep**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'heartbeat'
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'sleep' || null
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Subscribe to transaction data**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'subscribe',
        params: {
            subscription_id: subscription_id,
            last_mci: last_mci
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'subscribed'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Synchronous transaction data**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'catchup',
        params: {
            witnesses: witnesses,
            last_stable_mci: last_stable_mci,
            last_known_mci: last_known_mci
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {
            status: 'current'
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get the hash tree**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_hash_tree',
        params: {
            from_ball: from_ball,
            to_ball: to_bal      
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {
          "balls": [
            {
              "unit": "c2teVop7xa1BmH1LPvytvOKU8HHTrprGWQw+uHlWPHo=",
              "ball": "ht1QP48paWg5hpjx+Nbd/DSRFT8WlbQKk9+Uum0/tso=",
              "parent_balls": [
                "aEU1WiY9FQ9ihv9cKkX/EHWDxYYVaWYs2AL1yxyYZAQ="
              ]
            },
            {
              "unit": "tPbC6QLeweGuiGYRPsIhfd0TQWXWASYBJJUPGa9AOfw=",
              "ball": "ba5wFB+gewdRX6nSpxt6+Nt8PaRen4pLIYg3jC6EvIw=",
              "parent_balls": [
                "ht1QP48paWg5hpjx+Nbd/DSRFT8WlbQKk9+Uum0/tso="
              ]
            },
            {
              "unit": "J7quEBEZ5ksq+0vK4cku7NtWkZ4Kxb8aHCu66RkN2eU=",
              "ball": "4B+R+gW91BPmTqxF3G10oWmF4tg5LiHdpMQcL1ezqCo=",
              "parent_balls": [
                "ba5wFB+gewdRX6nSpxt6+Nt8PaRen4pLIYg3jC6EvIw="
              ]
            },
            ...
          ]
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get the main chain index**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'get_last_mci'
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 3795741
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Send message to client that is connected to hub

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/deliver',
        params: {
            to: device_address,
            pubkey: pubkey,
            signature: signature,
            encrypted_package: encrypted_message
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'accepted'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get temporary public key**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_temp_pubkey',
        params: permanent_pubkey
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'TEMP_PUBKEY_PACKAGE'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Update temporary public key**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/temp_pubkey',
        params: {
            temp_pubkey: temp_pubkey,
            pubkey: permanent_pubkey,
            signature: signature
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'updated'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Enable notifications**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/enable_notification',
        params: registrationId
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="request.json" %}
```
{
    type: 'response',
    content: {
        tag: tag,
        response: 'ok'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Disable notifications**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/disable_notification',
        params: registrationId
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'ok'
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get list of chat bots**

This requests returns response with chat bots of connected hub.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_bots'
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
[
  {
    "id": 29,
    "name": "Buy Bytes with Visa or Mastercard",
    "pairing_code": "A1i/ij0Na4ibEoSyEnTLBUidixtpCUtXKjgn0lFDRQwK@byteball.org/bb#0000",
    "description": "This bot helps to buy Bytes with Visa or Mastercard.  The payments are processed by Indacoin.  Part of the fees paid is offset by the reward you receive from the undistributed funds."
  },
  {
    "id": 31,
    "name": "World Community Grid linking bot",
    "pairing_code": "A/JWTKvgJQ/gq9Ra+TCGbvff23zqJ9Ec3Bp0XHxyZOaJ@byteball.org/bb#0000",
    "description": "Donate your device’s spare computing power to help scientists solve the world’s biggest problems in health and sustainability, and earn some Bytes in the meantime.  This bot allows you to link your Byteball address and WCG account in order to receive daily rewards for your contribution to WCG computations.\n\nWCG is an IBM sponsored project, more info at https://www.worldcommunitygrid.org"
  },
  {
    "id": 36,
    "name": "Username registration bot",
    "pairing_code": "A52nAAlO05BLIfuoZk6ZrW5GjJYvB6XHlCxZBJjpax3c@byteball.org/bb#0000",
    "description": "Buy a username and receive money to your @username instead of a less user-friendly cryptocurrency address.\n\nProceeds from the sale of usernames go to Byteball community fund and help fund the development and promotion of the platform."
  },
  ...
]
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get asset metadata**

This requests returns response with asset metadata \(unit and registry\). Example gets WCG Point asset metadata.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'hub/get_asset_metadata',
        params: 'IYzTSjJg4I3hvUaRXrihRm9+mSEShenPK8l8uKUOD3o='
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {
          "metadata_unit": "0xXOuaP5e3z38TF5ooNtDhmwNkh1i21rBWDvrrxKt0U=",
          "registry_address": "AM6GTUKENBYA54FYDAKX2VLENFZIMXWG",
          "suffix": null
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get transaction history**

This requests returns response with transaction history of specified addresses.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_history',
        params: {
            witnesses: witnesses,
            requested_joints: joints,
            addresses: addresses
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {Object}
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get chain link proofs**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_link_proofs',
        params: units
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: [Array]
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get the parent unit and the witness unit**

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_parents_and_last_ball_and_witness_list_unit',
        params: {
            witnesses: witnesses
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {
          "parent_units": [
            "QEdpDamxpez8dlOE4nF8TWwHs+efokBXlxQHK27/y4g="
          ],
          "last_stable_mc_ball": "xF0K/NKO6CGMU6XeBGXzurEcCajomfZMOAU4XmAy+6o=",
          "last_stable_mc_ball_unit": "R/RosYwuXJ/mXw5TTfWLVLHdtnnzaKn5EgJkDfzagcs=",
          "last_stable_mc_ball_mci": 3795817,
          "witness_list_unit": "J8QFgTLI+3EkuAxX+eL6a0q114PJ4h4EOAiHAzxUp24="
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### **Get specific attestation unit**

This requests returns response with attestation units for specified field and value.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_attestation',
        params: {
            attestor_address: 'FZP4ZJBMS57LYL76S3J75OJYXGTFAIBL',
            field: 'name',
            value: 'tarmo888'
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 'jeiBABcZI5fjyIPHkpb2PipLHzjUgafoPd0b6bdsGUI='
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Get all attestation data about address

This requests returns response with all the attestation data about specific address.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_attestations',
        params: {
            address: 'MNWLVYTQL5OF25DRRCR5DFNYXLSFY43K'
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: [
          {
            "unit": "jeiBABcZI5fjyIPHkpb2PipLHzjUgafoPd0b6bdsGUI=",
            "attestor_address": "FZP4ZJBMS57LYL76S3J75OJYXGTFAIBL",
            "profile": {
              "name": "tarmo888"
            }
          }
        ]
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Pick divisible inputs for amount

This requests returns response with spendable inputs for specified asset, amount and addresses.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/pick_divisible_coins_for_amount',
        params: {
            asset: '',
            addresses: addresses,
            last_ball_mci: last_ball_mci,
            amount: amount,
            spend_unconfirmed: 'own'
        }
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {Object}
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Get address definition

This requests returns response with address definition \(smart-contracts have address smart-contract definition only after somebody has spent from it\).

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_definition',
        params: 'JEDZYC2HMGDBIDQKG3XSTXUSHMCBK725'
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: [
          "sig",
          {
            "pubkey": "Aiy5z+jM1ySTZl1Qz1YZJouF7tU6BU++SYc/xe0Rj5OZ"
          }
        ]
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Get balances of addresses

This request returns a response with balances of one or more addresses.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_balances',
        params: [
            'JEDZYC2HMGDBIDQKG3XSTXUSHMCBK725',
            'UENJPVZ7HVHM6QGVGT6MWOJGGRTUTJXQ'
        ]
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: {
          "JEDZYC2HMGDBIDQKG3XSTXUSHMCBK725": {
            "base": {
              "stable": 1454617324,
              "pending": 2417784
            }
          },
          "UENJPVZ7HVHM6QGVGT6MWOJGGRTUTJXQ": {
            "base": {
              "stable": 0,
              "pending": 6
            }
          }
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Get profile unit IDs of addresses

It is possible for users to post a profile information about themselves, this request returns a response with all the profile data units.

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'light/get_profile_units',
        params: [
            'MNWLVYTQL5OF25DRRCR5DFNYXLSFY43K'
        ]
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: [
          "gUrJfmdDeYhTKAHM5ywydHXpvcensbkv8TPQLuPG3b0="
        ]
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Custom Request

You can add your own communication protocol on top of the Obyte one. See event [there](../list-of-events.md#event-for-custom-request).

{% code-tabs %}
{% code-tabs-item title="request.json" %}
```javascript
{
    type: 'request',
    content: {
        tag: tag,
        command: 'custom',
        params: params
    }
}
```
{% endcode-tabs-item %}

{% code-tabs-item title="response.json" %}
```javascript
{
    type: 'response',
    content: {
        tag: tag,
        response: 0 || 'some response' || {Object} || [Array]
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

