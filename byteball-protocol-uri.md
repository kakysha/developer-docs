---
description: >-
  This URI protocol enables developers to open the wallet app in desired screen
  with pre-filled inputs.
---

# byteball: protocol URI

If you want to open Obyte app for users then there is a `byteball:` protocol [URI ](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier#Examples)for that. If you are not aware what it is then it's like `mailto:` , `ftp:` , `tel:` or [intents on Android](https://developer.android.com/reference/android/content/Intent). When user visits a link with that protocol then it can open the wallet app in specific screen with pre-filled inputs, helping users to insert long wallet addresses or amounts without the need to copy-paste them.

## Open wallet for sending

We can [request payments with chat bot messages](writing-chatbots-for-byteball/#requesting-payments) and we can use the same commands as links on the website. This is how it will look as a hyperlink:

```markup
<a href="byteball:WALLET_ADDRESS?amount=123000&amp;asset=base">open Send screen</a>
```

`WALLET_ADDRESS` should be replaced with the wallet address where the funds should be sent, `amount` parameter should have `bigInt` type number in bytes \(not KByte, notMByte, not GByte\) and `asset` parameter is optional, by default it is `base` for bytes, but it can be issued [asset unit ID](issuing-assets-on-byteball.md#asset-id) too.

One website that heavily relies on this feature is [Currency converter for Obyte](https://tarmo888.github.io/bb-convert/) \([source code](https://github.com/tarmo888/bb-convert)\).

## Receiving textcoins via link

Textcoins are funds that can be sent via text messages. These text messages contain 12 randomly picked words and by entering them into the wallet app in exact same order, user will get the funds that were added to them. If you have ever received any textcoins then you also know that you don't actually have to re-type or copy-paste those words, textcoin receivers are usually directed to [Obyte textcoin claiming page](https://byteball.org/#textcoin?test-test-test-test-test-test-test-test-test-test-test-test), which has green "Recieve funds" button, which will launch the wallet app with exactly those textcoin words. Obyte textcoin claiming page tries to check first if user has wallet installed, but basically, underneath that button is a hyperlink similar to this:

```markup
<a href="byteball:textcoin?RANDOMLY-PICKED-TWELVE-WORDS">Recieve funds</a>
```

So, additionally to [sending textcoins with a bot](sending-textcoins-with-bot.md), you can also make clickable textcoins on your website \(link either to textcoin claiming page or directly to `byteball:` protocol\). Just make sure each user on your webiste has access to textcoins specifically generated for them. Otherwise, if all users see the same textcoins then it can become a rush to who claims the textcoins first.

## Pairing with another device

Bot Store on Obyte is like Apple App Store or Google Play Store, but instead of apps, Bot Store has chat bots that are written for Obyte platform. Each Obyte Hub can have each own bots that are shown for users of that app. Currently, most users are using the official wallet app and official Hub, but it makes sense that if somebody forks the wallet, they would use their own Hub for it too. This means that in the future, in order to get your chatbot exposed to maximum amount of users, it needs to get listed on all Hubs.

Luckily, adding chat bots to your wallet app is much easier on Obyte than it is to sideload apps on iOS or Android. All you need to do is add the pairing code that bot outputs to your website - everyone who has Byteball app already installed and clicks on it, will get automatically paired with your chatbot. Hyperlink to pairing code would be something like this:

```markup
<a href="byteball:AnpzF9nVTV5JZXzlG2fSnA+8UmjFuBdbqU+rJchz3qcN@byteball.org/bb#0000">Add Sports Betting bot</a>
```

One example, where all the official Hub bots are displayed on a website can be seen on [Bots section byteball.co](https://byteball.co/bots).

Pairing codes in hyperlinks are not limited to only chat bots, you could create a pairing link to any device, all you need to do is find you pairing invitation code \(Chat &gt; Add new device &gt; Invite other device\) and add the code after `byteball:` protocol inside the hyperlink.

## Sending commands to chat bots

Once you have paired with a chat bot already, you might wonder whether it is possible to get users from your website to some specific state in chat bot \(for example, by [sending a predefined command](writing-chatbots-for-byteball/#predefined-chat-commands)\) with a click on any hyperlink. There are 2 options how to do that, first option would be to fill the `pairing_secrets` database table with all the possible commands, but if there could be indefinite number of valid commands then easiest would be to [accept any pairing secret](writing-chatbots-for-byteball/#accept-any-pairing-secret) and use them as custom commands.

Websites that use this feature are [BB Odds](https://bb-odds.herokuapp.com/) and [Polls section on byteball.co](https://byteball.co/polls). How it can be done can be seen from Poll bot \([source code](https://github.com/byteball/poll-bot/blob/master/poll-bot.js)\). For example, opening a poll app with results of specific poll can be done with hyperlink like this:

```markup
<a href="byteball:AhMVGrYMCoeOHUaR9v/CZzTC34kScUeA4OBkRCxnWQM+@byteball.org/bb#stats-HAXKXC1EBn8GtiAiW0xtYLAJiyV4V1jTt1rc2TDJ7p4=">see stats</a>
```

Steem Attestation bot uses this method also as an alternative way how to refer other people, source code for that is more advanced that the poll bot code, but [it can be found on Github](https://github.com/byteball/steem-attestation/blob/66e72d4062f7c1f5a8a057366023c5eaa6863bf4/attestation.js#L90).

## Using protocol URI in QR code

`byteball:` protocol is not only for websites, it could be used in physical world too with QR code. This means you if users have Obyte app installed and they scan any QR code that contains any of the above codes \(just value of the href, without quotes and without the HTML around them\) then installed Obyte app will open the same way. QR code is not just meant to be printed on paper, it can work on websites too, giving the users the ability to use your services cross-device \(browse the website on laptop, scan QR code with phone and complete payment on phone\).

There are many online QR code generators to create those QR codes manually, but QR codes can be created in real-time too. Following is the example how to create a QR code with `byteball:` protocol link using jQuery.

```markup
<div class="container">
	<div id="qr_code"></div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.3.1/jquery.min.js" integrity="sha256-FgpCb/KJQlLNfOu91ta32o/NMZxltwRo8QtmkMRdAu8=" crossorigin="anonymous"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery.qrcode/1.0/jquery.qrcode.min.js" integrity="sha256-9MzwK2kJKBmsJFdccXoIDDtsbWFh8bjYK/C7UjB1Ay0=" crossorigin="anonymous"></script>
<script type="text/javascript">
$(document).ready(function() {
	var wallet_address = 'WALLET_ADDRESS';
	var byte_amount = 123000;
	var asset_unit = 'base';
	$('#qr_code').html('').qrcode({
		width: 512,
		height: 512,
		text: 'byteball:'+ wallet_address +'?amount='+ byte_amount +'&asset=' + asset_unit;
	});
});
</script>
```

A website that creates QR codes this way is [Currency converter for Obyte](https://tarmo888.github.io/bb-convert/) \([source code](https://github.com/tarmo888/bb-convert)\) and Steem Attestation bot \([source code](https://github.com/byteball/steem-attestation/blob/master/public/qr/index.html)\) generates referral links this way. Alternatively, there is also code example [how to generate QR code like that on server side](tutorials-for-newcomers/log-in-on-website-with-byteball.md).

