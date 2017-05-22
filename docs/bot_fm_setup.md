# Facebook Messenger Setup

If you have created a new bot for Facebook Messenger, you must configure your Facebook App and Page so that Coldbrew Bots API can properly connect to Messenger Platform and send/receive messages for your app.

## Step 1. Create Bot

Open [API Bot][apibotlink], and, find "Create Bot" button in "Manage Bots" menu.

<img src="https://git.io/vHIkP" width="200">
<img src="https://git.io/vHIkj" width="200">

Enter the name of your bot, and, you will see your new bot in the "Manage Bots" menu. _(Try swipe to the right if you don't see it.)_

## Step 2. Prepare Facebook App and Page

All Facebook Messenger bots require both Facebook App and Facebook Page. _It's not a Coldbrew Bots requirement._ Therefore, you will have to create a new Facebook App and Facebook Page for your bot, unless you already have them.

- [Create a Facebook Page][createfbpage]
- [Create a Facebook App][createfbapp] (Click `+ Add a New App` button)

## Step 3. Enter Page Access Token

After creating your app, you need to add "Messenger" product to your app, if you haven't yet.

Find the "Token Generation" sectionin "Messenger" product menu, and, select your Facebook Page to get the Page Access Token.

<img src="https://git.io/vHT54" width="400">

You need to set the Page Access Token for your bot using [API Bot][apibotlink]. Click "Page Access Token" button of the bot you created, and, enter the Page Access Token.

<img src="https://git.io/vHIt1" width="200">

## Step 4. Configure Webhooks

_IMPORTANT: You must complete the Step 3 first. Otherwise your subscription request during this step will fail._

Now find the Webhooks section of "Messenger" product menu, and, click "Setup Webhooks" button.

_(If this is your first time configuring webhooks, you will see "Callback URL" and "Verify Token" fields. Otherwise, you will have to go to "Webhooks" product menu to modify them.)_

<img src="https://git.io/vHIte" width="400">

- Callback URL: `https://bots.coldbrewcloud.com/fm/webhook/{your bot ID}`
- Verify Token: `{your bot verify token}`
- Subscription Fields: check "messages" and "messaging_postbacks" options

To view your bot's Webhook setup information, go back to [API Bot][apibotlink] again, then find "Webhook Setup Info" button in "Manage Bots" menu.

<img src="https://git.io/vHItr" width="200">

After entering those input fields, make sure to click "Verify and Save" button in the dialog.

<img src="https://git.io/vHIq8" width="400">

## Step 5. Configure Page Subscription

Now select the Facebook Page in the lower part of "Webhooks" section again, and, click "Subscribe" button.

<img src="https://git.io/vHT5G" width="400">

Once subscribed successfully, it will look like this:

<img src="https://git.io/vHT5l" width="400">

---

And, that's it. Now your bot is properly configured and connected to receive and send messages through [Coldbrew Bots API](api_reference.md). You can

[apibotlink]: https://www.messenger.com/t/260871171047071
[createfbpage]: https://www.facebook.com/pages/create/
[createfbapp]: https://developers.facebook.com/apps/
