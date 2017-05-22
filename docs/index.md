# Coldbrew Bots

...

## Features

- Cross Platform: [Facebook Messenger](https://developers.facebook.com/docs/messenger-platform) and [Slack](https://slack.com/apps/category/At0MQP5BEF-bots) bots available _(more to come)_
- Semantic Chat Bots: [session](#sessions)-based messaging and [natural language processing](https://wit.ai/) integration
- [Flow Engine](flow.md): programmable bot interaction engine
- [Google Analytics](https://analytics.google.com/): painless integration to map sessions, states, and events to GA

_NOTE: Coldbrew Bots is currently the Beta release. And, some features are currently open to the invited developers only._

## API Bot

[API Bot][apibotlink] is our own Facebook Messenger chat bot we created using [Coldbrew Bots API](api_reference.md) and [Flow Engine](flow.md). At first it was built as a demo for Coldbrew Bots, but, we soon realized that we could use it to serve our developers _(like yourself)_!

With [API Bot][apibotlink], you can do things like this:

- Create and manage your bots
- Manage your API tokens
- Access Bot Store _(coming soon)_
- Send feedback to us

<img src="https://git.io/vHT7X" width="200">

[Try It Now!][apibotlink] _(Best viewed in Messenger mobile app)_

_NOTE: Actually you will have to use our API Bot no matter what, because we didn't create a web site (or a mobile app) that serves the functionalities above. Sorry._

## Getting Started

See [Facebook Messenger Setup](bot_fm_setup.md) to create and configure a new bot for the Facebook Messenger.

## Core Concepts

### Sessions

Your bot is supposed to interact with multiple end users _(or multiple groups of users)_ simultaneously. And, from the end user's perspective, they expect their conversation with the bot is completely private and isolated from other conversations of the bot.

To make such isolation simpler, Coldbrew Bots API provides sessions for all conversations. Whenever an end user initiates the conversation with your bot, Coldbrew Bots API will create a new session and assign the end user to that session. And, all messages that you receive and send through Coldbrew Bots API will use that session to maintain private and independent conversation states with the end user.

_NOTE: In the current release, we support 1-to-1 session type only, but, we're working to add group and individual-in-group session types._

Each session has the following attributes:

- **ID**: a unique string ID of the session
- **State**: a session can have a state at a time. It's up to you how to utilize this session state, but, typical use cases would be to track of conversation phase of individual end users, or, to limit the possible interaction scenarios with the end users. Session state can be modified whenever you send a message through Coldbrew Bots API. (See [SessionUpdate](api_reference.md#sessionupdate) and [SendRequest](api_reference.md#sendrequest).)
- **Key-Value Storage**: you can store multiple string-based key value pairs in the session. All data in the session storage is private and permanent _(unless you delete them)_. Session data can be updated or deleted using [SessionUpdate](api_reference.md#sessionupdate) in [SendRequest](api_reference.md#sendrequest).
- **Modify Index**: a session maintain the modify index to keep track of all changes to the session. Whenever you make a change to the session (state or key-value data), this modify index will be incremented. You can use this modify index when you need to perform atomic operations (such as "compare-and-swap"). (See [SessionUpdate](api_reference.md#sessionupdate).)

### Message Pulling vs. Webhook Pushes

When using Coldbrew Bots, you *pull* the messages, whereas the Facebook Messenger Platform "pushes" the messages to your webhook endpoints. Both approaches have pros and cons, but, our pulling model has the following advantages:

1. It simplifies the local bot development setup. You don't need the network tunneling tools (such as [ngrok](https://ngrok.com/)) because your bot application will not receive incoming Webhook calls directly. Instead Coldbrew Bots API will handle the incoming Webhook calls and store them in Coldbrew Bots so you can pull those messages whenever you can.

2. You bot application does not have to be a web server accepting HTTP requests _(unless you have real needs)_. This can simplify your bot code, deployment structure, and, security configurations. No inbounds connections, no need for load balancer(s), etc.

[apibotlink]: https://www.messenger.com/t/260871171047071
