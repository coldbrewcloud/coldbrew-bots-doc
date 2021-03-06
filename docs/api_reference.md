# API Reference

## Bots API

#### Authentication

All HTTP requests to Bots API endpoints must include the API token in the request header.

```
Authorization: Bearer <your_api_token>
```

You can get your API token using [API Bot][apibotlink].

#### Rate Limiting

For requests to Bots API endpoints, you can make up to 1200 calls per minute. That is 20 calls per second. You can see your current rate limit status by checking the following HTTP response headers:

- `X-RateLimit-Duration-Sec`: rate limiting time window in seconds. It's currently set to `60`.
- `X-RateLimit-Limit`: the maximum number of calls you can make. It's currently set to `1200` for Free tier users.
- `X-RateLimit-Remaining`: the number of requests remaining in the current rate limit window.
- `X-RateLimit-Reset`: the time at which the current rate limit window resets in UTC epoch seconds.

And, if you exceed the rate limits, the endpoint will fail with the HTTP status code of `429`.

#### Message Queue Session Locking

When the client tries to retrieve messages using [Receive Messages](#receive-messages) endpoint, the Coldbrew Bots API will send as many messages as possible, but, it includes only one message per session. The client can continue to retrieve messages, but, the next message from the same session will be available after the client sends back the sending message(s) of the session. This feature is called Message Queue Session Locking and is enabled by default.

Why does it hold the messages and send a message per session? That is purely to help the client to process incoming messages sequentially (in the order of receipt) and to simplify the session state management in the client side. Without this feature, the client needs to make sure that it processes the incoming messages sequentially to avoid the conflicting session states _(which is very difficult to debug later)_.

If your application does not really care about the order of messages, or, if your application can handle the concurrent messages properly _(likey using `seq` field)_, you can simply disable this feature by including `nolock=1` query of your requests.

**IMPORTANT**: Unless you're 100% sure about your use case, we recommend not to disable this feature. Session locking greatly improves the consistency and predictability of your bot application.

As a safe measure, the API will automatically unlock the holding sessions after 5 seconds if it does not receive any outgoing (sending) messages with the same session ID, meaning you will receive another message from the same session about 5 seconds later even if you don't send any messages to that session.

### Receive Messages

```
GET https://bots.coldbrewcloud.com/bots/{bot_id}/messages
GET https://bots.coldbrewcloud.com/bots/{bot_id}/messages?nolock=1
```

This endpoint will return an array of messages that your bot had received. Currently the API will return up to 20 messages at a time.

**IMPORTANT**: Please be careful not to exceed the API rate limit. To be most responsive to the end users, it's important for your Bot to check the messages as fast as possible, but, you still need to balance between receive calls and send calls if you're expecting lots of messages. If not, most of time, the receive calls will get `404 Not Found`, and, it's probably a good idea to put slightly longer delay before making the next call.

Parameters:

- `bot_id`: bot ID
- `nolock=1`: you can disable the [session locking](#message-queue-session-locking) feature by including `nolock=1` in your request query. Otherwise the feature is always enabled by default.
- _(No HTTP request body)_

HTTP Response:

- `200`: response body will contain [ReceiveResponse](#receiveresponse).
- `404`: no messages received
- `400`: invalid bot ID
- `403`: permission error (most likely authorization issue)
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/messages"
```

### Send a Message

```
POST https://bots.coldbrewcloud.com/bots/{bot_id}/messages
```

Parameters:

- `bot_id`: bot ID
- HTTP request body: [SendRequest](#sendrequest)

HTTP Response:

- `200`: message was sent successfully
- `400`: invalid bot or session ID
- `403`: permission error (most likely authorization issue)
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    -XPOST -d '{"session_id": "841ba10b12d24427aa3b96ebb3a2ba9d","contents": [{"text": "hello there!"}]}' \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/messages"
```

### Push Messages

```
POST https://bots.coldbrewcloud.com/bots/{bot_id}/push
```

This endpoint lets you send the message to multiple users at once. Once the request is validated, the actual message send will be executed asynchronously. You will get `200 OK` response immediately but the API will start sending messages to the users (identified by their session IDs).

**IMPORTANT**: Currently there are additional limits on this push endpoint. One can send messages up to 1000 users at once, and, you can invoke this push request up to 6 times per hour.

**IMPORTANT**: To push messages to the end user who haven't talked to your bot more than 24 hours, you will need an approval for `pages_messaging_subscriptions` permission.

Parameters:

- `bot_id`: bot ID
- HTTP request body: [PushRequest](#pushrequest)

HTTP Response:

- `200`: message was sent successfully
- `400`: invalid bot, empty contents, empty session IDs, or, duplicate session IDs
- `403`: permission error (most likely authorization issue)
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    -XPOST -d '{"session_ids": ["841ba10b12d24427aa3b96ebb3a2ba9d", "27d4f5f2359a4b3891e595b9691036e1"],"contents": [{"text": "hello there!"}]}' \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/push"
```


### Bot Storage

Bot Storage is a key-value store that you can use to store the bot-level data. This is very similar to the session key-value storage except that this is bot-wide storage while the session storage is maintained at each individual session level.

#### Get All Keys

```
GET https://bots.coldbrewcloud.com/bots/{bot_id}/kv
```

Parameters:

- `bot_id`: bot ID

HTTP Response:

- `200`: returns an array of strings that contains the key names (e.g. `["key1", "key2", "key3"]`)
- `400`: invalid bot, empty contents, empty session IDs, or, duplicate session IDs
- `403`: permission error (most likely authorization issue)
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/kv"
```

#### Get a Key-Value

```
GET https://bots.coldbrewcloud.com/bots/{bot_id}/kv/{key}
```

Parameters:

- `bot_id`: bot ID
- `key`: key name

HTTP Response:

- `200`: returns the value in the response body
- `400`: invalid bot, empty contents, empty session IDs, or, duplicate session IDs
- `403`: permission error (most likely authorization issue)
- `404`: key not found
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/kv/key1"
```

#### Set a Key-Value

```
PUT https://bots.coldbrewcloud.com/bots/{bot_id}/kv/{key}
```

Parameters:

- `bot_id`: bot ID
- `key`: key name
- `X-Bot-KV-Modify-Index` _(Header)_: Optional HTTP header param. If you specify the modify index, the SET operation will be executed only if the current modify index is the same as this value. Otherwise it will return `409` error status. If you don't specify the modify index, the SET operation will be always executed (possibly overwriting other concurrent writes).
- HTTP request body: plain string value for the key

HTTP Response:

- `200`: a key-value pair was successfully created or updated
- `400`: invalid bot, empty contents, empty session IDs, or, duplicate session IDs
- `403`: permission error (most likely authorization issue)
- `409`: modify index conflict
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    -XPUT -d 'value' \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/kv/key1"
```

#### Delete a Key-Value

```
DELETE https://bots.coldbrewcloud.com/bots/{bot_id}/kv/{key}
```

Parameters:

- `bot_id`: bot ID
- `key`: key name
- `X-Bot-KV-Modify-Index` _(Header)_: Optional HTTP header param. If you specify the modify index, the DELETE operation will be executed only if the current modify index is the same as this value. Otherwise it will return `409` error status. If you don't specify the modify index, the DELETE operation will be always executed (possibly overwriting other concurrent writes).

HTTP Response:

- `200`: a key-value pair was successfully deleted
- `400`: invalid bot, empty contents, empty session IDs, or, duplicate session IDs
- `403`: permission error (most likely authorization issue)
- `409`: modify index conflict
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    -XDELETE \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/kv/key1"
```

### Get User Profile

```
GET https://bots.coldbrewcloud.com/bots/{bot_id}/users/{user_id}
```

Parameters:

- `bot_id`: bot ID
- `user_id`: user ID. You normally get the user ID from `sender_id` of [ReceivedMessage](#receivedmessage).
- _(No HTTP request body)_

HTTP Response:

- `200`: response body will contain [GetUserResponse](#getuserresponse).
- `404`: no user found
- `400`: invalid bot ID
- `403`: permission error (most likely authorization issue)
- `429`: API rate limit exceeded
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/users/77bef519d3474ecda828a124dd1426d5"
```

## Data Models

### ReceiveResponse

```json
{
    "messages": [
        {
            "sender_id": "fm-1558084600893191",
            "seq": 317,
            "contents": [
                {
                    "text": {
                        "text":"hello, world!"
                    }
                }
            ],
            "received_at": "2017-05-21T17:49:22.401-07:00",
            "session_id": "841ba10b12d24427aa3b96ebb3a2ba9d",
            "session_state": "main_menu",
            "session_kv": {
                "key1": "value1",
                "key2":"value2"
            },
            "session_modify_index": 311
        }        
    ]
}
```

- `messages`: an array of [ReceivedMessage](#receivedmessage)

### ReceivedMessage

```json
{
    "sender_id": "fm-1558084600893191",
    "seq": 317,
    "contents": [
        {
            "text": {
                "text":"hello, world!"
            }
        }
    ],
    "received_at": "2017-05-21T17:49:22.401-07:00",
    "session_id": "841ba10b12d24427aa3b96ebb3a2ba9d",
    "session_state": "main_menu",
    "session_kv": {
        "key1": "value1",
        "key2":"value2"
    },
    "session_modify_index": 311
}
```

- `sender_id`: user ID of message sender
- `seq`: sequential order index in session
- `contents`: an array of [ReceivableContent](#receivablecontent)
- `received_at`: time the message was originally received from the platform (Messenger)
- `session_id`: session ID (see [Sessions](index.md#sessions))
- `session_state`: the current session state value (see [Sessions](index.md#sessions))
- `session_kv`: session key-value storage (string dictionary) (see [Sessions](index.md#sessions))
- `session_modify_index`: the current modify index of the session (see [Sessions](index.md#sessions))


### ReceivableContent

Supported receivable content types: [TextContent](#textcontent), [MediaContent](#mediacontent), [ActionContent](#actioncontent), [PaymentContent](#paymentcontent), and, [ReferralContent](#referralcontent).

#### TextContent

```json
{
    "text": "hello world!",
    "quick_reply_payload": ""
}
```

- `text`: text message
- `quick_reply_payload`: _(optional)_ contains the payload data if end user clicked one of the quick reply button.

#### MediaContent

```json
{
    "type": 1,
    "url": "http://www.example.com/image.jpg"
}
```

- `type`: media type
    - `1`: image
    - `2`: video
    - `3`: audio
    - `4`: file
- `url`: URL of the media

#### ActionContent

```json
{
    "type": "messenger_postback",
    "payload": "tutorial_start"
}
```

- `type`: action type
    - `"messenger_get_started"`: _(Facebook Messenger only)_ the end user clicked "Get Started" button
    - `"messenger_postback"`: _(Facebook Messenger only)_ received a postback

#### PaymentShippingAddress

```json
{
    "street_1": "1 Hacker Way",
    "street_2": "",
    "city": "MENLO PARK",
    "state": "CA",
    "country": "US",
    "postal_code": "94025"
}
```

#### PaymentRequestedUserInfo

```json
{
    "shipping_address": {
        "street_1": "1 Hacker Way",
        "street_2": "",
        "city": "MENLO PARK",
        "state": "CA",
        "country": "US",
        "postal_code": "94025"
    },
    "contact_name": "Peter Chang",
    "contact_email": "peter@anemailprovider.com",
    "contact_phone": "+15105551234"
}
```

#### PaymentCredential

```json
{
    "provider_type": "stripe",
    "charge_id": "ch_18tmdBEoNIH3FPJHa60ep123",
    "fb_payment_id": "123456789"
}
```

- `provider_type`: payment provider; either `"stripe"` or `"paypal"`
- `charge_id`: provider charge ID
- `fb_payment_id`: a Facebook issued payment ID

#### PaymentAmount

```json
{
    "currency": "USD",
    "amount": "2.99"
}
```

- `currency`: payment currency
- `amount`: payment amount

#### PaymentContent

```json
{
    "payload": "user_payload",
    "requested_user_info": {
        "shipping_address": {
            "street_1": "1 Hacker Way",
            "street_2": "",
            "city": "MENLO PARK",
            "state": "CA",
            "country": "US",
            "postal_code": "94025"
        },
        "contact_name": "Peter Chang",
        "contact_email": "peter@anemailprovider.com",
        "contact_phone": "+15105551234"
    },
    "payment_credential": {
        "provider_type": "stripe",
        "charge_id": "ch_18tmdBEoNIH3FPJHa60ep123",
        "fb_payment_id": "123456789"
    },
    "amount": {
        "currency": "USD",
        "amount": "2.99"
    }
}
```

- `payload`: meta data defined in Buy button as payload
- `requested_user_info`: information that was requested from the user by the Buy button. See [PaymentRequestedUserInfo](#paymentrequesteduserinfo)
- `payment_credential`: payment credentials. See [PaymentCredential](#paymentcredential).
- `amount`: total amount of transaction. See [PaymentAmount](#paymentamount).

#### ReferralContent

```json
{
    "ref": "user defined referral payload",
    "source": 1,
    "type": 1,
    "first_time_use": true
}
```

- `ref`: user defined referral payload
- `source`: referral source
    - `1`: m.me short link
    - `2`: Ads
- `type`: referral type
    - `1`: m.me short link open thread
- `first_time_use`: whether the referral was included in Facebook Messenger Get Started event

See [here](https://developers.facebook.com/docs/messenger-platform/referral-params) for more detail.

### SendRequest

```json
{
    "session_id": "841ba10b12d24427aa3b96ebb3a2ba9d",
    "contents": [
        {
            "text": "hello there!"
        },
    ],
    "session_update": {
        "state": "new_state",
        "kv": {
            "key1": "new_value1",
            "key2": null
        }
    }
}
```

- `session_id`: you must set the same `session_id` value from [ReceivedMessage](#receivedmessage).
- `contents`: an array of [SendableContent](#sendablecontent)
- `session_update`: _(optional)_ if set, Coldbrew Bots API will update the session state, key-value storage while processing the send request. (See [SessionUpdate](#sessionupdate))

### SendableContent

Supported sendable content types: [SendableTextContent](#sendabletextcontent), [SendableMediaContent](#sendablemediacontent), [SendableListContent](#sendablelistcontent), [SendableMenuContent](#sendablemenucontent),

#### SendableTextContent

```json
{
    "text": "foo bar",
    "quick_replies": [],
    "buttons": []
}
```

- `text`: text message to send
- `quick_replies`: _(optional)_ an array of [SendableQuickReply](#sendablequickreply)
- `buttons`: _(optional)_ an array of [SendableButton](#sendablebutton)

**Note**: You cannot set both `quick_replies` and `button` at the same time.

#### SendableMediaContent

```json
{
    "type": 1,
    "url": "http://www.example.com/image.jpg"
}
```

- `type`: media type
    - `1`: image
    - `2`: video
    - `3`: audio
    - `4`: file
- `url`: URL of the media

#### SendableListContent

_(Facebook Messenger only)_ List content presents a set of items vertically.

```json
{
    "elements": [
        {
            "title": "Classic T-Shirt",
            "subtitle": "100% Cotton, 200% Comfortable",
            "image_url": "https://example.com/img/white-t-shirt.png",
            "button": {
                "type": 1,
                "title": "View Item",
                "payload": "http://example.com/item-detail"
            },
            "element_url": "http://example.com/more-info"
        },
        {
            "title": "Classic T-Shirt",
            "subtitle": "100% Cotton, 200% Comfortable",
            "image_url": "https://example.com/img/white-t-shirt.png",
            "button": {
                "type": 1,
                "title": "View Item",
                "payload": "http://example.com/item-detail"
            },
            "element_url": "http://example.com/more-info"
        }
    ],
    "large_top_element": true,
    "bottom_button": {
        "type": 1,
        "title": "Documentation",
        "payload": "http://www.example.com/docs"
    }
}
```

- `elements`: an array of [SendableListElement](#sendablelistelement)
- `large_top_element`: _(optional)_ if `true`, the first element will appear prominently above the other items.
- `button_button`: _(optional)_ a small button shown under all other items

Example:

<img src="https://git.io/vHT7Q" width="400">

#### SendableListElement

```json
{
    "title": "Classic T-Shirt",
    "subtitle": "100% Cotton, 200% Comfortable",
    "image_url": "https://example.com/img/white-t-shirt.png",
    "button": {
        "type": 1,
        "title": "View Item",
        "payload": "http://example.com/item-detail"
    },
    "element_url": "http://example.com/more-info"
}
```

- `title`: primary text
- `subtitle`: secondary text
- `image_url`: _(optional)_ URL of the image. (This is required for the first element if `large_top_element` is set to `true`.)
- `button`: _(optional)_ a button (See [SendableButton](#sendablebutton))
- `element_url`: _(optional)_ end user will open this URL when clicking the element itself (not the button).

#### SendableMenuContent

_(Facebook Messenger only)_ Menu content can be used to send a horizontal scrollable carousel of items, each composed of an image attachment, short description and buttons to request input from the user.

```json
{
    "elements": [
        {
            "title": "Classic T-Shirt",
            "subtitle": "100% Cotton, 200% Comfortable",
            "image_url": "https://example.com/img/white-t-shirt.png",
            "buttons": [
                {
                    "type": 2,
                    "title": "View Item",
                    "element_url": "view_item"
                },
                {
                    "type": 2,
                    "title": "Delete Item",
                    "element_url": "delete_item"
                },
            ],
            "element_url": "http://example.com/more-info"
        }
    ],
    "image_aspect_ratio": 1
}
```

- `elements`: an array of [SendableMenuElement](#sendablemenuelement)
- `large_top_element`: if `true`,

Example:

<img src="https://git.io/vHT7d" width="300">

#### SendableMenuElement

```json
{
    "title": "Classic T-Shirt",
    "subtitle": "100% Cotton, 200% Comfortable",
    "image_url": "https://example.com/img/white-t-shirt.png",
    "buttons": [
        {
            "type": 2,
            "title": "View Item",
            "element_url": "view_item"
        },
        {
            "type": 2,
            "title": "Delete Item",
            "element_url": "delete_item"
        },
    ],
    "element_url": "http://example.com/more-info"
}
```

- `title`: primary text
- `subtitle`: secondary text
- `image_url`: _(optional)_ URL of the image
- `element_url`: _(optional)_ end user will open this URL when clicking the element itself (not the button).
- `buttons`: an array of [SendableButton](#sendablebutton)

#### PaymentSummary

<img src="https://scontent-lax3-1.xx.fbcdn.net/v/t39.2365-6/14130025_327619210923721_919324010_n.png?oh=3919e8ea28666d2038994c5735c89786&oe=59DFD8F3" width="600">

```json
{
    "currency": "USD",
    "is_test_payment": true,
    "payment_type": 0,
    "merchant_name": "Coldbrew Cloud",
    "requested_user_info": ["contact_name", "contact_email"],
    "price_list": [
        {
            "label": "Item #1",
            "amount": "2.99"
        },
    ]
}
```

- `currency`: payment currency
- `is_test_payment`: if this is test payment
- `payment_type`:
    - `0`: fixed amount payment
    - `1`: flexible amount payment
- `merchant_name`: merchant name
- `requested_user_info`: an array of requested user informations
    - `"contact_name"`: name
    - `"contact_phone"`: phone number
    - `"contact_email"`: email address
    - `"shipping_address"`: shipping address
- `price_list`: an array of each individual items
    - `label`: item label
    - `amount`: item price

See [here](https://developers.facebook.com/docs/messenger-platform/send-api-reference/buy-button) for more details.

#### SendableButton

```json
{
    "type": 2,
    "title": "Cancel",
    "payload": "do_cancel",
    "share_contents": {},
    "payment_summary": {}
}
```

- `type`: button type
    - `1`: URL link
    - `2`: _(Facebook Messenger only)_ Messenger postback
    - `3`: _(Facebook Messenger only)_ Share
    - `4`: _(Facebook Messenger only)_ Buy (payment) button
- `title`: text shown on the button
- `payload`: _(Facebook Messenger only)_ _(optional)_ postback payload data (if `type` is `2`)
- `share_contents`: _(Facebook Messenger only)_ _(optional)_ sharing contents for Share button. [SendableMenuContent](#sendablemenucontent) type supported.
- `payment_summary`: _(Facebook Messenger only)_ _(optional)_ payment configuration. See [PaymentSummary](#paymentsummary).

#### SendableQuickReply

```json
{
    "title": "Confirm",
    "payload": "do_confirm",
    "image_url": "http://www.example.com/image.png"
}
```

- `title`: text shown on the quick reply button
- `payload`: _(Facebook Messenger only)_ postback payload data
- `image_url`: _(optional)_ URL of the image

Example:

<img src="https://git.io/vHT7N" width="300">

### SessionUpdate

```json
{
    "state": "new_state",
    "kv": {
        "key1": "new_value1",
        "key2": null
    },
    "modify_index": 311
}
```

- `state`: _(optional)_ if set, Coldbrew Bots API will try to change the state of the session.
- `kv`: _(optional)_ key-values that need to be updated or deleted (value set to `null`). Keys not included in this map will not be updated or deleted.
- `modify_index` _(optional)_ if set, the session update request will succeed only if the current session modify index matches this value. Otherwise all send request will fail with `409 Conflict` status. You can use this to perform "Compare-and-Swap" style operations.

See [Sessions](index.md#sessions) for more information.

### SendResponse

```json
{
    "send_results": [
        {
            "ok": true,
        },
        {
            "ok": false,
            "error_code": 112,
            "message": "some message"
        }
    ]
}
```

- `send_result`: an array of [OperationResult](#operationresult), each element represents the send operation result of corresponding [SendableContent](#sendablecontent) in [SendRequest](#sendrequest) in the same order.

### PushRequest

```json
{
    "session_ids": [
        "841ba10b12d24427aa3b96ebb3a2ba9d",
        "c533e179af87423f8bf70c829d0d9342",
        "27d4f5f2359a4b3891e595b9691036e1"
    ],
    "contents": [
        {
            "text": "hello there!"
        },
    ],
}
```

- `session_ids`: an array of session IDs
- `contents`: an array of [SendableContent](#sendablecontent)

### OperationResult

```json
{
    "ok": false,
    "error_code": 112,
    "message": "error message"
}
```

- `ok`: whether the operation was successful or not
- `error_code`: _(optional)_ numeric error code in case of error
- `message`: _(optional)_ text message about the operation

### GetUserResponse

```json
{
    "first_name": "Jon",
    "last_name": "Snow",
    "profile_pic": "https://files.coldbrewcloud.com/46e7308d31f14285b5e9ef0545d607b9",
    "locale": "en_US",
    "timezone": -7,
    "gender": "male"
}
```

- `first_name`: first name
- `last_name`: last name
- `profile_pic`: URL of the profile picture
- `locale`: user's locale
- `timezone`: time zone (number relative to GMT)
- `gender`: gender

[apibotlink]: https://m.me/coldbrewbots
