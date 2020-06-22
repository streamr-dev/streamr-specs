# Protocol

## Data Types

Our protocol is a JSON protocol. This means that we have the following types at our disposal: `string`, `number`, `object`, `array`, `boolean` and `null`. In the following, all `number` are positive integers or zero.

## Layers

The Streamr Protocol is made of three layers:
- **Network Layer:** Responsible for end-to-end unicast/multicast/broadcast communication primitives in the peer-to-peer network. Encapsulates messages from the Control Layer and defines other messages specific to communication between nodes that clients shouldn't be concerned with.
- **Control Layer:** Defines the control messages allowing communication entities to publish, subscribe, resend, etc... These messages are encapsulated by the Network Layer.
- **Message Layer:** Some messages in the Control Layer carry messages published in streams. The Message Layer defines the format of these message payloads, consisting of data and metadata of the messages.

This documentation describes the messages in each layer.

## Table of contents
- [Network Layer](#network-layer)
    - [StatusMessage](#statusmessage)
    - [InstructionMessage](#instructionmessage)
    - [FindStorageNodesMessage](#findstoragenodesmessage)
    - [StorageNodesMessage](#storagenodesmessage)
    - [WrapperMessage](#wrappermessage)
- [Control Layer](#control-layer)
    - [PublishRequest](#publishrequest)
    - [SubscribeRequest](#subscriberequest)
    - [UnsubscribeRequest](#unsubscriberequest)
    - [ResendLastRequest](#resendlastrequest)
    - [ResendFromRequest](#resendfromrequest)
    - [ResendRangeRequest](#resendrangerequest)
    - [BroadcastMessage](#broadcastmessage)
    - [UnicastMessage](#unicastmessage)
    - [SubscribeResponse](#subscriberesponse)
    - [UnsubscribeResponse](#unsubscriberesponse)
    - [ResendResponseResending](#resendresponseresending)
    - [ResendResponseResent](#resendresponseresent)
    - [ResendResponseNoResend](#resendresponsenoresend)
    - [ErrorResponse](#errorresponse)
- [Message Layer](#message-layer)
    - [StreamMessage](#streammessage)
    - [MessageID](#messageid)
    - [MessageRef](#messageref)
    - [GroupKeyRequest](#groupkeyrequest)
    - [GroupKeyResponse](#groupkeyresponse)
    - [GroupKeyReset](#groupkeyreset)
    - [GroupKeyErrorResponse](#groupkeyerrorresponse)


## Network Layer

All messages of the Network Layer are transmitted as JSON arrays with the following fields: `[version, type, source, ...typeSpecificFields]`.
`version` describes the version of the Network Layer. `type` is an integer to identify the message type according to the following table: 

messageType | Description
----------- | -----------
0 | StatusMessage
1 | InstructionMessage
2 | FindStorageNodesMessage
3 | StorageNodesMessage
4 | WrapperMessage

`source` is a string to identify the node that sent the message.

### StatusMessage

TODO: description of the purpose and usage of this message type.

```
[version, type, source, status]
```
Example:
```
["4.4.3", 0, "sender", "status"]
```

Field    | Type | Description
-------- | ---- | --------
`status` | `string` | TODO

### InstructionMessage

TODO: description of the purpose and usage of this message type.

```
[version, type, source, streamId, nodeAddresses]
```
Example:
```
["4.4.3", 1, "sender", "stream-id", ["address1", "address2"]]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | TODO
`nodeAddresses` | `array` | TODO

### FindStorageNodesMessage

TODO: description of the purpose and usage of this message type.

```
[version, type, source, streamId]
```
Example:
```
["4.4.3", 2, "sender", "stream-id"]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | TODO

### StorageNodesMessage

TODO: description of the purpose and usage of this message type.

```
[version, type, source, streamId, nodeAddresses]
```
Example:
```
["4.4.3", 1, "sender", "stream-id", ["address1", "address2"]]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | TODO
`nodeAddresses` | `array` | TODO

### WrapperMessage

Encapsulates messages of the Control Layer.

```
[version, type, source, controlLayerPayload]
```
Example:
```
// This encapsulates a SubscribeRequest (See in Control Layer section)
["4.4.3", 4, "sender", [2, 9, "stream-id", 0, "my-session-token"]]
```

Field    | Type | Description
-------- | ---- | --------
`controlLayerPayload` | ControlMessage | The array representation of the encapsulated `ControlMessage` (See Control Layer).

## Control Layer

This document describes version `2` of the Control Layer protocol.

All messages of the Control Layer are transmitted as JSON arrays and have the following structure with certain fields common to all messages: `[version, type, requestId, ...typeSpecificFields]`. 

Field       | Type      | Description
----------- | --------- | -----------
`version`   | `number`  | The protocol version of the Control Layer
`type`      | `number`  | An integer identifying the message type (see the table below)
`requestId` |  `string` | Identifies a particular `Request` message. The `requestId` can be chosen freely for each request, as long as it's unique within the context of a particular connection. The same `requestId` value gets echoed back in response messages that result from the request.

The `...typeSpecificFields` means that there are additional fields depending on message type. The possible message `type`s are:

messageType | Description
----------- | -----------
0 | BroadcastMessage
1 | UnicastMessage
2 | SubscribeResponse
3 | UnsubscribeResponse
4 | ResendResponseResending
5 | ResendResponseResent
6 | ResendResponseNoResend
7 | ErrorResponse
8 | PublishRequest
9 | SubscribeRequest
10 | UnsubscribeRequest
11 | ResendLastRequest
12 | ResendFromRequest
13 | ResendRangeRequest

The individual types are described in the remainder of this section. We start by describing the requests and then the responses.

### Requests sent

#### PublishRequest

Publishes a new message to a stream. Requires a write permission to the stream. Authentication requires the session token to be set. It contains a `StreamMessage` as a payload at the Message Layer level. The `StreamMessage` representation is also an array (nested in the `PublishRequest` array) which is described in the [StreamMessage](#streammessage) section.

```
[version, type, requestId, streamMessage, sessionToken]
```
Example:
```
[2, 8, "request-id", [...streamMessageFields], "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamMessage`| StreamMessage | The array representation of the `StreamMessage` to publish. Defined in the Message Layer.
`sessionToken` | `string` | User's session token retrieved with some authentication method.

#### SubscribeRequest

Requests that the client be subscribed to a stream-partition. Will result in a `SubscribeResponse` message, and a stream of `BroadcastMessage` as they are published.

```
[version, type, requestId, streamId, streamPartition, sessionToken]
```
Example:
```
[2, 9, "request-id", "stream-id", 0, "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id to subscribe to.
`streamPartition` | `number` | Partition id to subscribe to. Optional, defaults to 0.
`sessionToken` | `string` | User's session token retrieved with some authentication method. Optional. Public streams can be subscribed to without authentication.

#### UnsubscribeRequest

Unsubscribes the client from a stream-partition. The response message is `UnsubscribeResponse`.

```
[version, type, requestId, streamId, streamPartition]
```
Example:
```
[2, 10, "request-id", "stream-id", 0]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id to unsubscribe.
`streamPartition` | `number` | Partition id to unsubscribe. Optional, defaults to 0.

#### ResendLastRequest

Requests a resend of the last N messages for a stream-partition. Responses are either a sequence of `ResendResponseResending`, one or more `UnicastMessage`, and a `ResendResponseResent`; or a `ResendResponseNoResend` if there is nothing to resend.

```
[version, type, requestId, streamId, streamPartition, numberLast, sessionToken]
```
Example:
```
[2, 11, "request-id", "stream-id", 0, 500, "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id of the messages to resend.
`streamPartition` | `number` | Partition id of the messages to resend. Optional, defaults to 0.
`numberLast` | `number` | Resend the latest `numberLast` messages.
`sessionToken` | `string` | User's session token retrieved with some authentication method. Not required for public streams.

#### ResendFromRequest

Requests a resend, for a subscription id, of all messages of a particular publisher on a stream-partition, starting from a particular message defined by its reference. It carries a `MessageRef` payload at the Message Layer level, its array representation is described in the [MessageRef](#messageref) section. Responses are either a sequence of `ResendResponseResending`, one or more `UnicastMessage`, and a `ResendResponseResent`; or a `ResendResponseNoResend` if there is nothing to resend.

```
[version, type, requestId, streamId, streamPartition, fromMsgRef, publisherId, sessionToken]
```
Example:
```
[2, 12, "request-id", "stream-id", 0, [...msgRefFields], "publisher-id", "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id of the messages to resend.
`streamPartition` | `number` | Partition id of the messages to resend. Optional, defaults to 0.
`msgRef` | MessageRef | The array representation of the `MessageRef` to resend from. Defined in the Message Layer.
`publisherId` | `string` | The publisher id of the messages to resend. Can be `null` to resend the messages of all publishers.
`sessionToken` | `string` | User's session token retrieved with some authentication method. Not required for public streams.

#### ResendRangeRequest

Requests a resend, for a subscription id, of a range of messages of a particular publisher on a stream-partition between two message references. It carries two `MessageRef` payloads at the Message Layer level, described in the [MessageRef](#messageref) section. Responses are either a sequence of `ResendResponseResending`, one or more `UnicastMessage`, and a `ResendResponseResent`; or a `ResendResponseNoResend` if there is nothing to resend.

```
[version, type, requestId, streamId, streamPartition, fromMsgRef, toMsgRef, publisherId, msgChainId, sessionToken]
```
Example:
```
[2, 13, "request-id", "stream-id", 0, [...fromMsgRefFields], [...toMsgRefFields], "publisher-id", "msg-chain-id", "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id of the messages to resend.
`streamPartition` | `number` | Partition id of the messages to resend. Optional, defaults to 0.
`fromMsgRef` | MessageRef | The array representation of the `MessageRef` of the first message to resend. Defined in the Message Layer.
`toMsgRef` | MessageRef | The array representation of the `MessageRef` of the last message to resend. Defined in the Message Layer.
`publisherId` | `string` | Optional. Set both `publisherId` and `msgChainId` to resend only messages in a particular message chain. Set to `null` to resend messages for all publishers. 
`msgChainId` | `string` | Optional. Set both `publisherId` and `msgChainId` to resend only messages in a particular message chain. Set to `null` to resend messages for all publishers.
`sessionToken` | `string` | User's session token retrieved with some authentication method. Not required for public streams.

### Responses sent

#### BroadcastMessage

A message addressed to all subscriptions listening on the stream. It contains a `StreamMessage` as a payload at the Message Layer level. The `StreamMessage` representation is also an array (nested in the `BroadcastMessage` array) which is described in the [StreamMessage](#streammessage) section.

```
[version, type, requestId, streamMessage]
```
Example:
```
[2, 0, "request-id", [...streamMessageFields]]
```

Field    | Type | Description
-------- | ---- | --------
`streamMessage` | StreamMessage | The array representation of the `StreamMessage` to be broadcast. Defined in the Message Layer.

#### UnicastMessage

A message addressed in response to a specific resend request. It contains a `StreamMessage` as a payload at the Message Layer level. The `StreamMessage` representation is also an array (nested in the `UnicastMessage` array) which is described in the [StreamMessage](#streammessage) section.

```
[version, type, requestId, streamMessage]
```
Example:
```
[2, 1, "request-id", [...streamMessageFields]]
```

Field    | Type | Description
-------- | ---- | --------
`streamMessage` | StreamMessage | The array representation of the `StreamMessage` to be delivered. Defined in the Message Layer.

#### SubscribeResponse

Sent in response to a `SubscribeRequest`. Lets the client know that streams were subscribed to.

```
[version, type, requestId, streamId, streamPartition]
```
Example:
```
[2, 2, "request-id", "stream-id", 0]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id subscribed.
`streamPartition` | `number` | Partition id subscribed. Optional, defaults to 0.

#### UnsubscribeResponse

Sent in response to an `UnsubscribeRequest`.

```
[version, type, requestId, streamId, streamPartition]
```
Example:
```
[2, 3, "request-id", "stream-id", 0]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id unsubscribed.
`streamPartition` | `number` | Partition id unsubscribed. Optional, defaults to 0.

#### ResendResponseResending

Sent in response to a `ResendRequest`. Informs the client that a resend is starting.

```
[version, type, requestId, streamId, streamPartition]
```
Example:
```
[2, 4, "request-id", "stream-id", 0]
```

Field    | Type | Description
-------- | ---- | --------
`streamId`| `string` | Stream id to resend on.
`streamPartition` | `number` | Partition id to resend on. Optional, defaults to 0.

#### ResendResponseResent

Informs the client that a resend for a particular subscription is complete.

```
[version, type, requestId, streamId, streamPartition]
```
Example:
```
[2, 5, "request-id", "stream-id", 0]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id of the completed resend.
`streamPartition` | `number` | Partition id of the completed resend. Optional, defaults to 0.

#### ResendResponseNoResend

Sent in response to a `ResendRequest`. Informs the client that there was nothing to resend.

```
[version, type, requestId, streamId, streamPartition]
```
Example:
```
[2, 6, "request-id", "stream-id", 0]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id of resend not executed.
`streamPartition` | `number` | Partition id of the resend not executed. Optional, defaults to 0.

#### ErrorResponse

Sent in case of an error.

```
[version, type, requestId, errorMessage, errorCode]
```
Example:
```
[2, 7, "request-id", "error-message", "ERROR_CODE"]
```

Field    | Type | Description
-------- | ---- | --------
`errorMessage` | `string` | Human-readable message describing the error.
`errorCode` | `string` | Machine-readable string describing the type of error.

## Message Layer

The Message Layer contains three different types: `MessageID`, `MessageRef` and `StreamMessage`. This document describes message layer version `31`.

### StreamMessage

Contains the data and metadata for a message produced/consumed on a stream. It is a payload at the Control Layer for the following message types: `PublishRequest`, `BroadcastMessage`, `UnicastMessage`. Where `msgId` uniquely identifies the `StreamMessage` and is the array representation of the `MessageID` defined [below](#messageid). `prevMsgRef` allows to identify the previous `StreamMessage` on the same stream and same partition published by the same producer. It is used to detect missing messages. It is the array representation of the `MessageRef` defined [below](#messageref).

```
[version, msgId, prevMsgRef, contentType, encryptionType, content, signatureType, signature]
```
Example:
```
[31, [...msgIdFields], [...msgRefFields], 27, 0, "contentData", 1, "0x29c057786Fa..."]
```

Field    | Type | Description
-------- | ---- | --------
`version` | `number` | Is currently 30.
`msgId` | MessageID |Array representation of the `MessageID` to uniquely identify this message. 
`prevMsgRef` | MessageRef | Array representation of the `MessageRef` of the previous message on a message chain (defined in the `msgId`). Used to detect missing messages.
`contentType` | `number` | Determines how the content should be parsed according to the table below.
`encryptionType` | `number` | Encryption type as defined by the table below.
`content` | `string` | Content data of the message.
`signatureType` | `number` | Signature type as defined by the table below.
`signature` | `string` | Signature of the message, signed by the producer. Encoding depends on the signature type.

#### `contentType`

`contentType` | Description
-------------- | --------
27 | Normal message. The `content` is a string containing a valid JSON object.
28 | [GroupKeyRequest](#groupkeyrequest)
29 | [GroupKeyResponse](#groupkeyresponse)
30 | [GroupKeyReset](#groupkeyreset)
31 | [GroupKeyErrorResponse](#groupkeyerrorresponse)

#### `signatureType`

`signatureType` | Name | Description | Signature payload fields to be concatenated in order
-------------- | ---- |------------ | -----------------------
0 | `NONE` | No signature. signature field is empty in this case. | None.
1 | `ETH_LEGACY` | Ethereum signature produced by old clients (Message Layer version 29). The signature field is encoded as a hex string. | `streamId`, `streamPartition`, `timestamp`, `publisherId`, `content`
2 | `ETH` | Ethereum signature produced by current clients (since Message Layer version 30). The signature field is encoded as a hex string. | all the `msgId` fields, (`streamId`, `streamPartition`, `timestamp`, `sequenceNumber`, `publisherId`, `msgChainId`), `prevMsgRef`, `content`

#### `encryptionType`

`encryptionType` | Description
-------------- | --------
0 | Unencrypted, plaintext message.
RSA | Content is asymetrically encrypted using RSA.
AES | Content is symmetrically encrypted using AES.
NEW_KEY_AND_AES | The content contains a concatenation of a new AES key and the content, encrypted with the old key.

### MessageID

Uniquely identifies a `StreamMessage`.

```
[streamId, streamPartition, timestamp, sequenceNumber, publisherId, msgChainId]
```
Example:
```
["stream-id", 0, 425354887214, 0, "0xAd23Ba54d26D3f0Ac057...", "msg-chain-id"]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id the corresponding `StreamMessage` belongs to.
`streamPartition` | `number` | Stream partition the `StreamMessage` belongs to.
`timestamp` | `number` | Timestamp of the `StreamMessage` (milliseconds format).
`sequenceNumber` | `number` | Sequence number of the `StreamMessage` within the same timestamp. Defaults to 0.
`publisherId` | `string` | Id of the publisher of the `StreamMessage`. Must be an Ethereum address if the `StreamMessage` has an Ethereum signature (`signatureType` = 1).
`msgChainId` | `string` | Id of the message chain this `StreamMessage` is part of. This message chain id is chosen by the publisher and defined locally for the `streamId`-`streamPartition`-`publisherId` triplet.

### MessageRef

Used inside a `StreamMessage` to identify the previous message on the same `msgChainId` (defined above).

```
[timestamp, sequenceNumber]
```
Example:
```
[425354887001, 0]
```

Field    | Type | Description
-------- | ---- | --------
`timestamp` | `number` | Timestamp of the `StreamMessage` published on the same stream and same partition by the same producer.
`sequenceNumber` | `number` | Sequence Number of the `StreamMessage`.

### GroupKeyRequest

Example of valid `content` for `contentType` 28:
```
{
  "requestId": "random-string-to-pair-requests-with-responses",
  "streamId": "id-of-stream-to-be-decrypted",
  "publicKey": "subscriber-rsa-public-key",
  "range": { // optional
    "start": 342546546,
    "end": 379080012
  }
}
```

### GroupKeyResponse

Example of valid `content` for `contentType` 29:
```
{
  "requestId": "random-string-to-pair-requests-with-responses", // repeat the requestId of the request
  "streamId": "id-of-stream-to-be-decrypted",
  "keys": [{
    "groupKey": "some-encrypted-group-key"
    "start": 342546000
  }, {
    "groupKey": "some-later-encrypted-group-key"
    "start": 369146000
  }]
}
```

### GroupKeyReset

Example of valid `content` for `contentType` 30:
```
{
  "streamId": "id-of-stream-to-be-reset",
  "groupKey": "new-encrypted-group-key"
  "start": 9086906
}
```

### GroupKeyErrorResponse

Example of valid `content` for `contentType` 31:
```
{
  "requestId": "random-string-to-pair-requests-with-responses", // repeat the requestId of the request
  "streamId": "id-of-stream-for-which-a-key-was-requested",
  "code": "ERROR_CODE",
  "message": "Example error message"
}
```
