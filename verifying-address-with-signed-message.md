---
description: >-
  If you need to verify that a user owns a particular address, you can do it by
  offering to sign a message.
---

# Verifying address with signed message

You can also verify ownership of address by requesting a payment from their address and this method is more appropriate if you need to receive a payment anyway.

To request the user to sign a message, send this message in chat:

```text
[any text](sign-message-request:message)
```

where `message` is the challenge to be signed. This request will be displayed in the user's wallet as a link that the user can click and confirm signing. Once the user signs, your chatbot receives a specially formatted message:

```text
[...](signed-message:base64_encoded_text)
```

You can parse and validate it with this code:

```javascript
// handle signed message
let arrSignedMessageMatches = text.match(/\(signed-message:(.+?)\)/);
if (arrSignedMessageMatches){
	let signedMessageBase64 = arrSignedMessageMatches[1];
	var validation = require('byteballcore/validation.js');
	var signedMessageJson = Buffer(signedMessageBase64, 'base64').toString('utf8');
	try{
		var objSignedMessage = JSON.parse(signedMessageJson);
	}
	catch(e){
		return null;
	}
	validation.validateSignedMessage(objSignedMessage, err => {
		if (err)
			return device.sendMessageToDevice(from_address, 'text', err);
		if (objSignedMessage.signed_message !== challenge)
			return device.sendMessageToDevice(from_address, 'text',
				"You signed a wrong message: "+objSignedMessage.signed_message+", expected: "+challenge);
		if (objSignedMessage.authors[0].address !== user_address)
			return device.sendMessageToDevice(from_address, 'text',
				"You signed the message with a wrong address: "+objSignedMessage.authors[0].address+", expected: "+user_address);
		
		// all is good, address proven, continue processing
	});
}
```

The above code also checks that the user signed the correct message and with the correct address.

`objSignedMessage` received from the peer has the following fields:

* `signed_message`: \(string\) the signed message
* `authors`: array of objects, each object has the following structure \(similar to the structure of `authors` in a unit\):
  * `address`: \(string\) the signing address
  * `definition`: \(array\) definition of the address
  * `authentifiers`: \(object\) signatures for different signing paths

To validate the message, call `validation.validateSignedMessage` as in the example above.

Note that the challenge that you offer to sign must be both clear to the user and sufficiently unique. The latter is required to prevent reuse of a previously saved signed message.

