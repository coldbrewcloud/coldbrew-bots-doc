# Coldbrew Bots

Coldbrew Bots is a chat bot development platform that simplifies

## Features

- Cross platform: currently [Facebook Messenger](https://developers.facebook.com/docs/messenger-platform) and [Slack](https://slack.com/apps/category/At0MQP5BEF-bots) bots available
- [Session](#sessions)-based messaging
- [Flow Engine](flow.md): Programmable bot engine
- Seamless [Natural Language Processing](https://wit.ai/) integration
- Painless [Google Analytics](https://analytics.google.com/) integration
- [Interactive API Bot][apibotlink]: Tutorials, dashboards,

_NOTE: Coldbrew Bots is currently the Beta release. And, some features are currently available to the invited developers only._

## Interactive API Bot

In the current release, we provide

This [API Bot][apibotlink] itself was built on [Coldbrew Bots API](api_reference.md) and [Flow Engine](flow.md).

<img src="https://files.coldbrewcloud.com/9feb52fba21e4388970d7d293e21aca2.PNG" width="200">

- Create and manage your bots
- Manage your API tokens
- Access Bot Store (coming soon)
- Try interactive tutorials
- Share feedback with us

[Try It Now!][apibotlink] _(Best viewed in Messenger mobile app)_

## How It Works

This is the typical workflow:

1. Create a Bot using [API Bot][apibotlink]
2. Connect the bot to your Facebook App and Page
3. Start receiving and sending messages via [Coldbrew Bots API](api_reference.md)

Once you create your bot via Coldbrew Bots, you can easily receive and send the messages with your end users. You will use these 2 API endpoints mostly:

- `GET https://bots.coldbrewcloud.com/bots/{bot_id}/messages` to receive messages
- `POST https://bots.coldbrewcloud.com/bots/{bot_id}/messages` to send messages

When using these endpoints, you must include your API token in `Authorization` HTTP header. An example curl command to receive a message will look like this:

```bash
curl -H "Authorization: Bearer <your_api_token>" "https://bots.coldbrewcloud.com/bots/<your_bot_id>/messages"
```

And this endpoint will return either a message (see `ReceiveResponse` model in [API Reference](api_reference.md)) or `404 Not Found` status if there's no message received.

See [API Reference](api_reference.md) for more information.

## Sessions

All messages received/sent via Coldbrew Bots API will have a session ID. Your bot will most likely interact with multiple end users at the same time. The concept of session lets you easily maintain isolated conversational status with different end users (or group of end users).

Each session will include these data attributes:

- Session ID: a unique ID for the session
- Session State: you can use the session state (string) to maintain different stages or phases of the conversation with different end users.
- Key-Value Storage: you can use session KV store to store information (e.g. answers to questions)
- Modify Index: whenever you modify the session, this index will increment. And you can use this modify to perform compare-and-swap operations. But that's optional.

## Other Topics



### Message Pulling vs. Webhook Pushes

In Coldbrew Bots API, you "pull" the messages, whereas the Facebook Messenger Platform "pushes" the messages to your webhook endpoints. Both have pros and cons, but, the pulling model has the following advantages:

1. It makes local bot app development simpler. you don't need the network tunneling tools (such as [ngrok](https://ngrok.com/)) because your bot applicaiton does not receive incoming webhook calls directly. Coldbrew Bots API will receive and store the webhook calls so your bot app can retrieve (pull) the messages later.

2. You bot app does not have to be a web server accepting HTTP requests. It makes the bot code and deployment structure much simpler. e.g. no inbound connections, no need for load balancers.

[apibotlink]: https://www.messenger.com/t/260871171047071
