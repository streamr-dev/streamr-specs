# Streamr Protocol

## Data Types

The Streamr Protocol is a JSON protocol. This means that we have the following types at our disposal: `string`, `number`, `object`, `array`, `boolean` and `null`. In the following, all `number` are positive integers or zero.

## Encoding

The JSON messages are UTF8 encoded when transported under an underlying binary protocol (which from the perspective of the message definitions is undefined, but in practice the network uses websocket and in the future WebRTC).  

## Layers

The Streamr Protocol is made of three layers:
- **Network Layer:** Responsible for end-to-end unicast/multicast/broadcast communication primitives in the peer-to-peer network. Encapsulates messages from the Control Layer and defines other messages specific to communication between nodes that clients shouldn't be concerned with.
- **Control Layer:** Defines the control messages allowing communication entities to publish, subscribe, resend, etc... These messages are encapsulated by the Network Layer.
- **Stream Layer:** Messages published to streams. The Stream Layer defines the format of these message payloads, consisting of data and metadata of the messages. Note that some messages in the Control Layer wrap Stream Layer messages. 

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
- [Stream Layer](#stream-layer)
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

type | Description
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

Publishes a new message to a stream. Requires a write permission to the stream. Authentication requires the session token to be set. It contains a [Stream Layer](#stream-layer) message as a payload.

```
[version, type, requestId, streamLayerMessage, sessionToken]
```
Example:
```
[2, 8, "request-id", [...streamMessage], "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamLayerMessage`| `array` | The [Stream Layer](#stream-layer) message to publish.
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

Requests a resend, for a subscription id, of all messages of a particular publisher on a stream-partition, starting from a particular message defined by its reference. It carries a [`MessageRef`](#messageref) to define the starting point for the resend. Responses are either a sequence of `ResendResponseResending`, one or more `UnicastMessage`, and a `ResendResponseResent`; or a `ResendResponseNoResend` if there is nothing to resend.

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
`msgRef` | MessageRef | The array representation of the `MessageRef` to resend from. Defined in the Stream Layer.
`publisherId` | `string` | The publisher id of the messages to resend. Can be `null` to resend the messages of all publishers.
`sessionToken` | `string` | User's session token retrieved with some authentication method. Not required for public streams.

#### ResendRangeRequest

Requests a resend, for a subscription id, of a range of messages of a particular publisher on a stream-partition between two message references. It carries two [MessageRef](#messageref)s to identify the start and end messages. Responses are either a sequence of `ResendResponseResending`, one or more `UnicastMessage`, and a `ResendResponseResent`; or a `ResendResponseNoResend` if there is nothing to resend.

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
`fromMsgRef` | MessageRef | The array representation of the `MessageRef` of the first message to resend. Defined in the Stream Layer.
`toMsgRef` | MessageRef | The array representation of the `MessageRef` of the last message to resend. Defined in the Stream Layer.
`publisherId` | `string` | Optional. Set both `publisherId` and `msgChainId` to resend only messages in a particular message chain. Set to `null` to resend messages for all publishers. 
`msgChainId` | `string` | Optional. Set both `publisherId` and `msgChainId` to resend only messages in a particular message chain. Set to `null` to resend messages for all publishers.
`sessionToken` | `string` | User's session token retrieved with some authentication method. Not required for public streams.

### Responses sent

#### BroadcastMessage

A message addressed to all subscriptions listening on the stream. It wraps a [Stream Layer](#stream-layer) message to be consumed by subscribers.

```
[version, type, requestId, streamLayerMessage]
```
Example:
```
[2, 0, "request-id", [...streamMessage]]
```

Field    | Type | Description
-------- | ---- | --------
`streamLayerMessage` | `array` | The [Stream Layer](#stream-layer) message being broadcasted.

#### UnicastMessage

A message addressed in response to a specific resend request. It wraps a [Stream Layer](#stream-layer) message to be consumed by the subscriber.

```
[version, type, requestId, streamLayerMessage]
```
Example:
```
[2, 1, "request-id", [...streamMessage]]
```

Field    | Type | Description
-------- | ---- | --------
`streamLayerMessage` | `array` | The [Stream Layer](#stream-layer) message being unicasted.

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

## Stream Layer

This document describes Stream Layer version `32`.

Stream Layer messages are the actual content in streams. They are identified by a [`MessageID`](#messageid), are chained together via a reference to the previous message, [`MessageRef`](#messageref) to establish message order, they are cryptographically signed by their publisher. The most prominent Stream Layer message is the [`StreamMessage`](#streammessage), which is used to transport arbitrary application-level messages from publishers to subscribers. Additionally, there are a few other message types used for requesting and delivering encryption keys.

It's worth noting that the following Control Layer messages contain a Stream Layer message: `PublishRequest`, `BroadcastMessage`, and `UnicastMessage`.

All Stream Layer messages have the following structure:

 ```
 [version, msgId, prevMsgRef, messageType, contentType, encryptionType, encryptionKeyId, content, signatureType, signature]
 ```
 
 Field        | Type      | Description
 ------------ | --------- | -----------
 `version`    | `number`  | The protocol version of the Stream Layer
 `msgId`      | [`MessageID`](#messageid) | The [`MessageID`](#messageid) that uniquely identifies this message.
 `prevMsgRef` | [`MessageRef`](#messageref) | The [`MessageRef`](#messageref) pointing to the previous message on a message chain. Optional. Used to detect missing messages.
 `messageType`| `number`  | Determines how the message should be handled. See the table below.
 `contentType`| `number`  | Determines how the (decrypted) content field should be parsed. For a list of possible values, see the table below.
 `encryptionType`| `number` | Encryption type as defined by the table below.
 `groupKeyId`| `string` | Identifies the symmetric (AES) key used by the publisher to encrypt the message content. `null` if the message is not AES encrypted.
 `content` | `string` | Content data of the message. Depends on the `messageType` how the content should be handled.
 `signatureType` | `number` | Signature type as defined by the table below.
 `signature` | `string` | Signature of the message, signed by the producer. Encoding depends on the signature type.

The various type fields have the following possible values:

#### `messageType`

`messageType` | Description
-------------- | --------
27 | [StreamMessage](#streammessage)
28 | [GroupKeyRequest](#groupkeyrequest)
29 | [GroupKeyResponse](#groupkeyresponse)
30 | [GroupKeyReset](#groupkeyreset)
31 | [GroupKeyErrorResponse](#groupkeyerrorresponse)
32 | [GroupKeyRotate](#groupkeyrotate)

#### `contentType`

Describes how the `content` should be parsed. 

Note that encrypted `content` is always a hex-encoded binary string. Once decrypted, the `content` should be interpreted based on `contentType`.

`contentType` | Description
------------- | -----------
0 | JSON

Other content types, including binary types, will be defined in the future.

#### `encryptionType`

`encryptionType` | Description
-------------- | --------
0 | Unencrypted, plaintext message.
1 | Content is asymetrically encrypted using RSA.
2 | Content is symmetrically encrypted using AES. The `groupKeyId` identifies the key used.

#### `signatureType`

`signatureType` | Name | Description | Signature payload fields to be concatenated in order
-------------- | ---- |------------ | -----------------------
0 | `NONE` | No signature. The `signature` field is null in this case. | None.
1 | `ETH_LEGACY` | Ethereum signature produced by old clients. The signature field is encoded as a hex string. | `streamId`, `streamPartition`, `timestamp`, `publisherId`, `content`
2 | `ETH` | Ethereum signature produced by current clients (since Stream Layer version 30). The signature field is encoded as a hex string. | `streamId`, `streamPartition`, `timestamp`, `sequenceNumber`, `publisherId`, `msgChainId`, `prevMsgRef.timestamp`, `prevMsgRef.sequenceNumber`, `content`

### StreamMessage

Contains an arbitrary, application-specific `content` payload, i.e. the `content` of this message should be passed to the subscribing application's message handler. This is the message type that applications interact with via publishing and subscribing to messages.

Example (unencrypted JSON message):
```
[32, [...msgIdFields], [...msgRefFields], 27, 0, 0, "{\"content\":42}", 2, "0x29c057786Fa..."]
```

Example (encrypted JSON message):
```
[32, [...msgIdFields], [...msgRefFields], 27, 0, 0, "{\"content\":42}", 2, "0x29c057786Fa..."]
```

### GroupKeyRequest

Sent by a subscriber to the publisher's key exchange stream in order to request a missing symmetric encryption key from the publisher. The protocol implementation should send this message when it encounters a message encrypted with a key that the subscriber doesn't have. The publisher should handle this message and respond with a `GroupKeyResponse` for successfully retrieved keys (if any) and a `GroupKeyErrorResponse` for keys that could not be recovered (if any).

Note that the publisher may have multiple instances connected to the network under the same `publisherId`. All of those instances will receive the `GroupKeyRequest` and respond individually, meaning that the requestor may subsequently receive many `GroupKeyResponse`s and `GroupKeyErrorResponse`s. The implementation should be prepared for receiving many responses for a `GroupKeyRequest`.

`GroupKeyRequest`s must be unencrypted (`encryptionType 0`). They contain no secrets. 

The `content` encoded as `contentType 0` (JSON) is as follows:

```
[requestId, streamId, rsaPublicKey, [groupKeyIds]]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `requestId`  | `string`  | A string to uniquely identify this request. Will be echoed back in the `GroupKeyResponse`.
 `streamId`   | `string`  | The stream for which a key is being requested.
 `rsaPublicKey`| `string` | The RSA public key of the requestor. The response will be encrypted for this public key.
 `groupKeyIds`| `array`   | An array of strings containing the `groupKeyIds` that are being requested.
 
Example of a `GroupKeyRequest` including the rest of the `Stream Layer` fields:

```
[32, [...msgIdFields], [...msgRefFields], 28, 0, 0, ["requestId", "streamId", "rsaPublicKey", ["keyId"]], 2, "0x29c057786Fa..."]
```

### GroupKeyResponse

Sent by a publisher as a success response to a `GroupKeyRequest`. The response is sent to the requestor's key exchange stream. The requestor should handle this message by adding the contained keys to their key storage, and re-attempting to decrypt any messages that previously could not be decrypted due to missing the keys. 

`GroupKeyResponse`s must be encrypted with (`encryptionType 1 (RSA)`) for the `rsaPublicKey` provided in the `GroupKeyRequest`. 

The `content` encoded as `contentType 0` (JSON) is as follows:

```
[requestId, streamId, [groupKeys]]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `requestId`  | `string`  | The `requestId` of the `GroupKeyRequest`, identifying which request is being responded to.
 `streamId`   | `string`  | The stream for which a key is being delivered.
 `groupKeys`  | `array`   | Array of `[groupKeyId, encryptedGroupKey]` pairs, containing the requested keys encrypted with RSA for the public key provided in the `GroupKeyRequest`. The `encryptedGroupKey` is a hex-encoded binary string.
 
Example of a `GroupKeyResponse` including the rest of the `Stream Layer` fields:

```
[32, [...msgIdFields], [...msgRefFields], 29, 0, 0, ["requestId", "streamId", ["keyId","123abc"]], 2, "0x29c057786Fa..."]
```

### GroupKeyReset

Sent by publishers to each subscribers' key exchange streams whenever they want to re-key the stream (in order to revoke access from some subscribers). 

The requestor should handle this message by adding the contained keys to their key storage.

`GroupKeyReset`s must be encrypted with (`encryptionType 1 (RSA)`) and publishers should send them to subscribers whose `rsaPublicKey` they are aware of (due to receiving a `GroupKeyRequest` earlier, for example. 

The `content` encoded as `contentType 0` (JSON) is as follows:

```
[streamId, groupKeyId, groupKey]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `streamId`   | `string`  | The stream for which a key is being delivered.
 `groupKeyId` | `string`  | The id of the `groupKey`.
 `groupKey`   | `string`  | The group key as a hex-encoded binary string.
 
Example of a `GroupKeyReset` including the rest of the `Stream Layer` fields:

```
[32, [...msgIdFields], [...msgRefFields], 30, 0, 0, ["keyId", "123abc"], 2, "0x29c057786Fa..."]
```

### GroupKeyErrorResponse

In case there's a problem retrieving the keys requested via a `GroupKeyRequest`, this message gets sent by a publisher to the requestor's key exchange stream. Such error situations might include, for example:

- The requested key might not be found in the publisher's key store
- The requestor no longer has permissions to access the stream
- The request was incorrectly formed 

The error response contains the list of keys for which the exchange failed, and an explanation why.

`GroupKeyErrorResponse`s are always unencrypted (`encryptionType 0`). The `content` encoded as `contentType 0` (JSON) is as follows:

```
[requestId, streamId, errorCode, errorMessage, [groupKeyIds]]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `requestId`  | `string`  | The `requestId` of the `GroupKeyRequest`, identifying which request is being responded to.
 `streamId`   | `string`  | The stream for which a key is being delivered.
 `errorCode`  | `string`  | An error code to help the subscriber categorize the error.
 `errorMessage`| `string` | An error message intended for a human, explaining what went wrong.
 `groupKeyIds`| `array`   | An array of strings containing the `groupKeyIds` that were requested but could not be retrieved due to the error.
 
Example of a `GroupKeyErrorResponse` including the rest of the `Stream Layer` fields:

```
[32, [...msgIdFields], [...msgRefFields], 31, 0, 0, ["requestId", "streamId", "ERROR_CODE", "Error message", ["groupKeyId1"]], 2, "0x29c057786Fa..."]
```

### GroupKeyRotate

This message is sent by publishers to rotate the encryption key and deliver it to existing subscribers (for forward secrecy). Subscribers should add the contained key into their key storage. The message is encrypted with another (previous) key. Publishers usually send this message to inform subscribers of a new key in advance, so that they every subscriber doesn't need to send a `GroupKeyRequest` to the publisher when they encounter the first `StreamMessage` encrypted with the new key.

`GroupKeyRotate`s must be encrypted (`encryptionType 2 (AES)`), and the immediately preceding key should be used to encrypt it. This allows subscribers who have the preceding key to decrypt and start using the new key. 

The `content` encoded as `contentType 0` (JSON) is as follows:

```
[groupKeyId, groupKey]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `groupKeyId` | `string`  | The id of the `groupKey`.
 `groupKey`   | `string`  | The group key as a hex-encoded binary string.
 
Example of a decrypted `GroupKeyRotate` including the rest of the `Stream Layer` fields:

```
[32, [...msgIdFields], [...msgRefFields], 32, 0, 0, ["keyId", "123abc], 2, "0x29c057786Fa..."]
```

### MessageID

Uniquely identifies a Stream Layer message.

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

Used inside a Stream Layer message to identify the previous message on the same `msgChainId` (defined above). If the `MessageRef` is not provided by the publisher, then the stream will have weaker delivery guarantees, as subscribers will not be able to detect missing messages and request gapfills.

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
