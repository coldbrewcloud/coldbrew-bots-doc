# Coldbrew Bots API

## Bots API

```
https://bots.coldbrewcloud.com/bots
```

**Authentication**

All HTTP requests to `/bots/` endpoints must include API Token in the request header.

```
Authorization: Bearer <your_api_token>
```

You can get your API token using [API Bot][apibotlink].

### Receive a Message

```
GET https://bots.coldbrewcloud.com/bots/{bot_id}/messages
```

Parameters:

- `bot_id`: bot ID
- _(No HTTP request body)_

HTTP Response:

- `200`: response body will contain [ReceiveResponse](#receiveresponse).
- `404`: no messages received
- `400`: invalid bot ID
- `403`: permission error (most likely authorization issue)
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
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    -XPOST -d '{"session_id": "841ba10b12d24427aa3b96ebb3a2ba9d","contents": [{"text": "hello there!"}]}' \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/messages"
```

### Get User Profile

```
GET https://bots.coldbrewcloud.com/bots/{bot_id}/users/{user_id}
```

Parameters:

- `bot_id`: bot ID
- `user_id`: user ID. You normally get the user ID from `sender_id` of [ReceiveResponse](#receiveresponse).
- _(No HTTP request body)_

HTTP Response:

- `200`: response body will contain [GetUserResponse](#getuserresponse).
- `404`: no user found
- `400`: invalid bot ID
- `403`: permission error (most likely authorization issue)
- `500`: other server side errors

Example request (curl):

```bash
curl -H "Authorization: Bearer 7d63da3eb2944e969eae3d9d5b036c1c" \
    "https://bots.coldbrewcloud.com/bots/3314b5adb4914a7e97be78c2b66e26c8/users/77bef519d3474ecda828a124dd1426d5"
```

## Models

### ReceiveResponse

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

Supported receivable content types: [TextContent](#textcontent), [MediaContent](#mediacontent), [EventContent](#eventcontent).

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

#### EventContent

```json
{
    "type": "messenger_postback",
    "payload": "tutorial_start"
}
```

- `type`: event type
    - `"messenger_get_started"`: _(Facebook Messenger only)_ the end user clicked "Get Started" button
    - `"messenger_postback"`: _(Facebook Messenger only)_ received a postback

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

- `session_id`: you must set the same `session_id` value from [ReceiveResponse](#receiveresponse).
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

<img src="https://files.coldbrewcloud.com/7768d8438538499ca0beff9fcde084fb.png" width="400">

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

<img src="https://files.coldbrewcloud.com/74a72d398896415d82a24c948eda7b30.PNG" width="300">

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

#### SendableButton

```json
{
    "type": 2,
    "title": "Cancel",
    "payload": "do_cancel"
}
```

- `type`: button type
    - `1`: URL link
    - `2`: _(Facebook Messenger only)_ Messenger postback
    - `3`: _(Facebook Messenger only)_ Share
- `title`: text shown on the button
- `payload`: _(Facebook Messenger only)_ _(optional)_ postback payload data (if `type` is `2`)

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

<img src="https://files.coldbrewcloud.com/66c98bbe49d74ac8b1b4fcc09a37793c.PNG" width="300">

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

See [Sessions](home.md#sessions) for more information.

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

[apibotlink]: https://www.messenger.com/t/260871171047071
