# Plugins

The Tezos Reward Distributor uses a plugin style subsystem for sending out payment notifications. If a message is labeled 'administrative style' it will contain information which are in general not meant for the end user of e.g. a staking service but for the adminstrators who monitor the quality of the service.

Each plugin and its configuration are detailed out below. Some plugins may require additional libraries that are not installed with TRD.

The configuration parameters for all plugins are located in the bakers .yaml config file. Please take a look at [the example config](https://github.com/tezos-reward-distributor-organization/tezos-reward-distributor/blob/master/examples/tz1boot1pK9h2BVGXdyvfQSv8kd1LQM6H889.yaml).

Individual plugins will not load if not properly configured.

You must specify which plugins to enable by adding their name to the "enabled" list. No plugins are enabled by default.

In this example, even though the webhook plugin is properly configured, it is *not* listed in the 'enabled' section, and therefor will not activate.

```
plugins:
  enabled:
  webhook:
    endpoint: https://mydomain.com/webhook.php
    token: Xynl6svphysd3BhjLP6IS
```

In order to activate the webhook plugin, you must add the name of the plugin to the *enabled* section as shown here:

```
plugins:
  enabled:
  - webhook
  webhook:
    endpoint: https://mydomain.com/webhook.php
    token: Xynl6svphysd3BhjLP6IS
```

If that doesn't make sense to you, please read the [YAML format documentation](https://yaml.org/spec/history/2001-05-26.html).

---

## Email Plugin

This plugin will send the configured recipients an 'administrative style' email of payouts status with a CSV report attached.

**NOTE**: The file `email.ini` has been deprecated and is no longer used.

### Parameters

* smtp_user: (Required unless no login is used) The username for SMTP authentication
* smtp_pass: (Required unless no login is used) The password or application token for SMTP authentication
* smtp_nologin: (Optional) true or false, set to true to send without login, it will work if recipients are local to the server you connect to.
* smtp_host: (Required) The host of your SMTP server
* smtp_port: (Required) The port for communication to your SMTP server. TLS uses 587.
* smtp_tls: (Optional) true/false. Only TLS is supported as SSL is usually deprecated in email servers.
* smtp_sender: (Required) The address for 'From' in the email
* smtp_sender_name: (Optional) String - You can specify the display name for the sender, put a name to the sender, instead of just having the email address
* smtp_recipients: (Required) A YAML list containing email addresses of intended recipients. Must be list format even if 1 recipient.

### Example Config

```
plugins:
  email:
    smtp_user: user@domain.com
    smtp_pass: horsebatterystaple2
    smtp_host: smtp.domain.com
    smtp_port: 587
    smtp_tls: true
    smtp_sender: trdnotice@domain.com
    smtp_recipients:
      - bob@domain.com
      - alice@hotmail.com
```

### Example 2 config
```
plugins:
  email:
    smtp_nologin: true
    smtp_host: smtp.domain.com
    smtp_port: 587
    smtp_tls: true
    smtp_sender: trdnotice@domain.com
    smtp_sender_name: "TRD Notification Source X"
    # when using nologin, you cannot relay externally, all recipients must be 
    # from a domain local to the server you connect with smtp_host.
    # in this case dmain.com is the server that is reponsible for domain.com email.
    smtp_recipients:
      - bob@domain.com
      - alice@domain.com
```

**Note about environment variables**:

To avoid the storage of sensitive values, smtp_user and smtp_pass can be defined as environment values. 

Environment variables will supersede yaml values.
* smtp_user will become SMTP_USER
* smtp_pass will become SMTP_PASS

## Telegram Plugin

This plugin allows payouts notifications to be sent via Telegram bot to specific chatIds, including groups.

You must first create a Telegram bot to generate the *bot_api_key* and you must discover your, or your groups', *chat_id*. There are many guides/tutorials on the internet for how to do this.

The Telegram plugin does not need read access to any messages and the bot will not respond to any commands. This is a "send only" style of bot.

This plugin supports two notification styles: admin notifications, and payouts notifications

### Parameters

* admin_chat_ids: A required YAML list containing chat IDs of users and/or groups to receive administrative-related messages. This might include failed payouts, or lack of funds messages.  Must be list format even if only 1 ID.
* public_chat_ids: An, optional, YAML list containing chat IDs of users and/or groups to receive completed payouts message as defined by `telegram_text`. Must be list format even if only 1 ID.
* bot_api_key: This is the API token that you get from @TheBotFather after creating your bot.
* telegram_text: This text will be sent to all chat ids list in 'payouts_chat_ids'. The text will be passed through a filter to replace the following placeholders: `%CYCLE%`, `%NDELEGATORS%`, `%TREWARDS%`. Basic HTML formatting is supported; see Telegram Docs for supported HTML tags. Emojis and other unicode characters are supported.

### Example Config

```
plugins:
  telegram:
    admin_chat_ids:
      - 123456789
      - 345112344
    payouts_chat_ids:
      - -13134455
      - 827384777
    bot_api_key: 988877766:SKDJFLSJDFJLJSKDFJLKSDJFLKJDF
    telegram_text: >
      &#x2728; Reward for cycle %CYCLE% <b>complete</b>! We had %NDELEGATORS% <i>delegators</i> in the cycle and paid out %TREWARDS% XTZ in rewards!
```

### Example Result

Admin Message:
![Admin Image](https://i.imgur.com/57Lim8s.png)

Payouts Message (using above example config text):
![Payouts Image](https://i.imgur.com/7ES9m2e.png)

## Twitter Plugin

This plugin allows payout notifications to be sent via a Twitter tweet. This plugin does not read existing tweets, or read any DMs. The plugin supports adding hashtags to your tweet. The plugin posts tweets "as you", meaning your @-handle will be the author of the post. No other information is submitted to the tweet.

Follow this outline to generate the API tokens and secrets for the plugin:

1. Sign up as a developer at https://developer.twitter.com/
2. Create a Twitter app. N.B. you will need to apply for [Elevated Access](https://developer.twitter.com/en/docs/twitter-api/getting-started/about-twitter-api#v2-access-level).
3. Enable Read and Write permissions on the app
4. Under 'Keys and Tokens' there are two sections: Consumer Keys, and Authentication Tokens
	* Under 'Consumer Keys', generate 'API Key and Secret'
	* Under 'Authentication Tokens', generate 'Access Token & Secret'
	* You can ignore 'Bearer Token'

### Example Config

**NOTE**: All 4 pieces of keys and tokens are required.

**NOTE**: `tweet_text` will be passed through a filter to replace the following placeholders: `%CYCLE%`, `%NDELEGATORS%`, `%TREWARDS%`

**NOTE**: The twitter plugin only supports payouts notifications; administrative notifications will not be sent via twitter plugin

```
plugins:
  twitter:
    api_key: XXXXXXXX
    api_secret: ZZZZZZZZ
    access_token: YYYYYYYY
    access_secret: WWWWWWWW
    tweet_text: >
      Reward for cycle %CYCLE% complete! We had %NDELEGATORS% delegators in the cycle and paid out %TREWARDS% XTZ in rewards. #ourbakery #rewards #tezos
```

The above example configuration will produce a tweet that looks like this:

```
Reward for cycle 290 complete! We had 133 delegators
in the cycle and paid out 1234.98 XTZ in rewards.
#ourbakery #rewards #tezos
```

## Discord Plugin

This plugin makes an HTTP POST request to a Discord Webhook URL.

You must be an admin of a discord server to generate a webhook URL. Do so by going to 'Server Settings' -> 'Integrations' -> 'New Webhook'. Select the name, upload an avatar icon (if desired), and select which channel the messages will appear.

Paste the generated URL into the appropriate field in the plugin section. You can customize the message for 'payouts' notifications. This is handled exactly as in the Twitter plugin.

### Example Config

* endpoint: The webhook URL generated by your discord server
* send_admin: [True|False] - Whether or not to send administrative-style messages using the bot. These messages will appear alongside general user-style messages.
* discord_text: This text will be sent to the channel configured for your bot. The text will be passed through a filter to replace the following placeholders: `%CYCLE%`, `%NDELEGATORS%`, `%TREWARDS%`. Markdown formatting is supported; Emojis and other unicode characters are supported.

```
plugins:
  enabled:
    - discord
  discord:
    endpoint: https://discord.com/api/webhooks/9876543212345678/OmAfadfasdfasdfasdfasdfasdfasfsdf
    send_admin: False
    discord_text: >
      Rewards for cycle %CYCLE% are completed.
      We paid out %TREWARDS% XTZ in rewards to %NDELEGATORS% delegators.        
```

### Example Result

![PayoutsDiscordImage](https://i.imgur.com/kjuiHSm.png)

## Webhook Plugin

This plugin makes an HTTP POST request to an endpoint. This endpoint will receive a JSON object containing the reward data.

For simple security, configure a random token (alphanumeric string) to be included in the root JSON object. Your receiver script should verify this token before accepting data.

Your script can return a short message in the response body. This will be displayed on the TRD console and written to the TRD logs.

**NOTE**: The webhook plugin supports administrative notifications which includes 'subject', 'message', and payouts records.

### Example JSON Object

"payouts" is a JSON array of objects, each object representing the status of a payout. All Tez amounts are in mutez.

```
{
  "timestamp": 1604982374,
  "token": "Xynl6svphysd3BhjLP6IS",
  "subject": "Payouts of cycle 22 completed",
  "message": "Much longer message example containing more information",
  "payouts": [
    {
      "address": "tz1LrHNbbCLgNJZsEsTUYFvWz2THgJC8fHyX",
      "paymentAddress": "tz1LrHNbbCLgNJZsEsTUYFvWz2THgJC8fHyX",
      "addressType": "D",
      "cycle": 415,
      "stakingBalance": 65116664916,
      "ratio": 0.16080255,
      "feeRatio": 0.01398283,
      "amount": 207756875,
      "feeAmount": 18065815,
      "feeRate": 0.08,
      "payable": true,
      "skipped": false,
      "opHash": "oo4Gikxyj8cMqM8xzgWxqnsxXoGwfhZqHDrLqpA6ENmMVoYUnVd",
      "neededActivation": false,
      "paymentStatus": "DONE"
    },
    {...}
  ]
}
```

### Example Config

```
plugins:
  webhook:
    endpoint: https://mydomain.com/webhook.php
    token: Xynl6svphysd3BhjLP6IS
```
