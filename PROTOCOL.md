# Streamr Protocol

## Data Types

The Streamr Protocol is a JSON protocol. This means that we have the following types at our disposal: `string`, `number`, `object`, `array`, `boolean` and `null`. In the following, all `number` are positive integers or zero.

## Encoding

The JSON messages are UTF8 encoded when transported under an underlying binary protocol (which from the perspective of the message definitions is undefined, but in practice the network uses websocket and in the future WebRTC).  

## Sub-Protocols

The Streamr Protocol consists of three sub-protocols:
- **Control Protocol:** messages that allow communication entities to publish, subscribe, resend, etc.
- **Stream Protocol:** messages published to streams and related entities. Note that some Control Protocol messages wrap Stream Protocol messages. 
- **Tracker Protocol:** messages exchanged between trackers and nodes. Allows nodes to find each other, as well as inform the tracker of their status. 

This documentation describes the messages and entities in each sub-protocol.

## Table of contents
- [Control Protocol](#control-protocol)
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
- [Stream Protocol](#stream-protocol)
    - [StreamMessage](#streammessage)
    - [MessageID](#messageid)
    - [MessageRef](#messageref)
    - [GroupKeyRequest](#groupkeyrequest)
    - [GroupKeyResponse](#groupkeyresponse)
    - [GroupKeyAnnounce](#groupkeyannounce)
    - [GroupKeyErrorResponse](#groupkeyerrorresponse)
- [Tracker Protocol](#tracker-protocol)
    - [StatusMessage](#statusmessage)
    - [InstructionMessage](#instructionmessage)
    - [StorageNodesRequest](sStoragenodesrequest)
    - [StorageNodesResponse](#storagenodesresponse)

## Control Protocol

This document describes version `2` of the Control Protocol.

All messages are transmitted as JSON arrays and have the following structure with certain fields common to all messages: `[version, type, requestId, ...typeSpecificFields]`. 

Field       | Type      | Description
----------- | --------- | -----------
`version`   | `number`  | The version of the Control Protocol
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

Publishes a new message to a stream. Requires a write permission to the stream. Authentication requires the session token to be set. It contains a [Stream Protocol](#stream-protocol) message as a payload.

```
[version, type, requestId, streamProtocolMessage, sessionToken]
```
Example:
```
[2, 8, "request-id", [...streamMessage], "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamProtocolMessage`| `array` | The [Stream Protocol](#stream-protocol) message to publish.
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
[version, type, requestId, streamId, streamPartition, fromMsgRef, publisherId, msgChainId, sessionToken]
```
Example:
```
[2, 12, "request-id", "stream-id", 0, [...msgRefFields], "publisher-id", "msg-chain-id" "my-session-token"]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | Stream id of the messages to resend.
`streamPartition` | `number` | Partition id of the messages to resend. Optional, defaults to 0.
`msgRef` | MessageRef | The array representation of the `MessageRef` to resend from. Defined in the Stream Protocol.
`publisherId` | `string` | The publisher id of the messages to resend. Can be `null` to resend the messages of all publishers.
`msgChainId` | `string` | Optional. Set both `publisherId` and `msgChainId` to resend only messages in a particular message chain. Set to `null` to resend messages for all publishers.
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
`fromMsgRef` | MessageRef | The array representation of the `MessageRef` of the first message to resend. Defined in the Stream Protocol.
`toMsgRef` | MessageRef | The array representation of the `MessageRef` of the last message to resend. Defined in the Stream Protocol.
`publisherId` | `string` | Optional. Set both `publisherId` and `msgChainId` to resend only messages in a particular message chain. Set to `null` to resend messages for all publishers. 
`msgChainId` | `string` | Optional. Set both `publisherId` and `msgChainId` to resend only messages in a particular message chain. Set to `null` to resend messages for all publishers.
`sessionToken` | `string` | User's session token retrieved with some authentication method. Not required for public streams.

### Responses sent

#### BroadcastMessage

A message addressed to all subscriptions listening on the stream. It wraps a [Stream Protocol](#stream-protocol) message to be consumed by subscribers.

```
[version, type, requestId, streamProtocolMessage]
```
Example:
```
[2, 0, "request-id", [...streamMessage]]
```

Field    | Type | Description
-------- | ---- | --------
`streamProtocolMessage` | `array` | The [Stream Protocol](#stream-protocol) message being broadcasted.

#### UnicastMessage

A message addressed in response to a specific resend request. It wraps a [Stream Protocol](#stream-protocol) message to be consumed by the subscriber.

```
[version, type, requestId, streamProtocolMessage]
```
Example:
```
[2, 1, "request-id", [...streamMessage]]
```

Field    | Type | Description
-------- | ---- | --------
`streamProtocolMessage` | `array` | The [Stream Protocol](#stream-protocol) message being unicasted.

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

## Stream Protocol

This document describes Stream Protocol version `32`.

Messages of Stream Protocol are the actual content in streams. They are identified by a [`MessageID`](#messageid), are chained together via a reference to the previous message, [`MessageRef`](#messageref) to establish message order, they are cryptographically signed by their publisher. The most prominent message is the [`StreamMessage`](#streammessage), which is used to transport arbitrary application-level messages from publishers to subscribers. Additionally, there are a few other message types used for requesting and delivering encryption keys.

It's worth noting that the following Control Protocol messages contain a Stream Protocol message: `PublishRequest`, `BroadcastMessage`, and `UnicastMessage`.

All Stream Protocol messages have the following structure:

 ```
 [version, msgId, prevMsgRef, messageType, contentType, encryptionType, groupKeyId, content, newGroupKey, signatureType, signature]
 ```
 
 Field        | Type      | Description
 ------------ | --------- | -----------
 `version`    | `number`  | The  version of the Stream Protocol
 `msgId`      | [`MessageID`](#messageid) | The [`MessageID`](#messageid) that uniquely identifies this message.
 `prevMsgRef` | [`MessageRef`](#messageref) | The [`MessageRef`](#messageref) pointing to the previous message on a message chain. Optional. Used to detect missing messages.
 `messageType`| `number`  | Determines how the message should be handled. See the table below.
 `contentType`| `number`  | Determines how the (decrypted) content field should be parsed. For a list of possible values, see the table below.
 `encryptionType`| `number` | Encryption type as defined by the table below.
 `groupKeyId`| `string` | Identifies the AES key used by the publisher to encrypt the message content. For AES encryption, the `groupKeyId` is a unique identifier chosen by the publisher. If the message is RSA encrypted (used in key exchange), this field contains the public key of the intended recipient. The field is `null` if the message is not encrypted.
 `content` | `string` | Content data of the message. Depends on the `messageType` how the content should be handled. In a [StreamMessage](#streammessage) the `content` is arbitrary, while in the other message types it follows a certain structure.
 `newGroupKey`| [`GroupKey`](#groupkey) | Used in key rotation to communicate a future key along with the message. The field is `null` if the message doesn't contain a new key.
 `signatureType` | `number` | Signature type as defined by the table below.
 `signature` | `string` | Signature of the message, signed by the publisher. A hex-encoded string.

The various type fields have the following possible values:

#### `messageType`

`messageType` | Description
-------------- | --------
27 | [StreamMessage](#streammessage)
28 | [GroupKeyRequest](#groupkeyrequest)
29 | [GroupKeyResponse](#groupkeyresponse)
30 | [GroupKeyAnnounce](#groupkeyannounce)
31 | [GroupKeyErrorResponse](#groupkeyerrorresponse)

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
1 | Content is asymmetrically encrypted using RSA. When using RSA, the `groupKeyId` is set to the public key of the recipient. 
2 | Content is symmetrically encrypted using AES. When using AES, the `groupKeyId` is set to a unique value freely chosen by the publisher, used to identify and refer to the key used.

#### `signatureType`

`signatureType` | Name | Description | Signature payload fields to be concatenated in order
-------------- | ---- |------------ | -----------------------
0 | `NONE` | No signature. The `signature` field is null in this case. | None.
2 | `ETH` | Ethereum signature produced by current clients (since Stream Protocol version 30). The signature field is a hex-encoded string. | `streamId`, `streamPartition`, `timestamp`, `sequenceNumber`, `publisherId`, `msgChainId`, `prevMsgRef.timestamp`, `prevMsgRef.sequenceNumber`, `content`, `newGroupKey` (fields with `null` values are simply skipped)

The concatenated payload is signed using the same [ECDSA and secp256k1 based method used in Ethereum](https://yos.io/2018/11/16/ethereum-signatures/).

### StreamMessage

Contains an arbitrary, application-specific `content` payload, i.e. the `content` of this message should be passed to the subscribing application's message handler. This is the message type that applications interact with via publishing and subscribing to messages.

Example (unencrypted JSON message):
```
[32, [...msgIdFields], [...msgRefFields], 27, 0, 0, null, "{\"foo\":42}", null, 2, "0x29c057786Fa..."]
```

Example (AES-encrypted JSON message):
```
[32, [...msgIdFields], [...msgRefFields], 27, 0, 2, "groupKeyId", "9abef2710b", null, 2, "0x29c057786Fa..."]
```

Example (AES-encrypted JSON message carrying an [EncryptedGroupKey](#encryptedgroupkey)):
```
[32, [...msgIdFields], [...msgRefFields], 27, 0, 2, "groupKeyId", "9abef2710b", "[\"newGroupKeyId\", \"encryptedGroupKey\"]", 2, "0x29c057786Fa..."]
```

### GroupKeyRequest

Sent by a subscriber to the publisher's key exchange stream in order to request a missing symmetric encryption key from the publisher. The protocol implementation should send this message when it encounters a message encrypted with a key that the subscriber doesn't have. The publisher should handle this message and respond with a `GroupKeyResponse` for successfully retrieved keys (if any) and a `GroupKeyErrorResponse` for keys that could not be recovered (if any).

Note that the publisher may have multiple instances connected to the network under the same `publisherId`. All of those instances will receive the `GroupKeyRequest` and respond individually, meaning that the requestor may subsequently receive many `GroupKeyResponse`s and `GroupKeyErrorResponse`s. The implementation should be prepared for receiving many responses for a `GroupKeyRequest`.

`GroupKeyRequest`s must be unencrypted (`encryptionType=0`). They contain no secrets. 

The `content` encoded with `contentType=0` (JSON) is as follows:

```
[requestId, streamId, rsaPublicKey, [groupKeyIds]]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `requestId`  | `string`  | A string to uniquely identify this request. Will be echoed back in the `GroupKeyResponse`.
 `streamId`   | `string`  | The stream for which a key is being requested.
 `rsaPublicKey`| `string` | The RSA public key of the requestor. The response will be encrypted for this public key.
 `groupKeyIds`| `array`   | An array of strings containing the `groupKeyIds` that are being requested.
 
Example of a `GroupKeyRequest` including the rest of the `Stream Protocol` fields:

```
[32, [...msgIdFields], [...msgRefFields], 28, 0, 0, null, "[\"requestId\", \"streamId\", \"rsaPublicKey\", [\"keyId\"]]", null, 2, "0x29c057786Fa..."]
```

### GroupKeyResponse

Sent by a publisher as a success response to a `GroupKeyRequest`. The response is sent to the requestor's key exchange stream. The requestor should handle this message by adding the contained keys to their key storage, and re-attempting to decrypt any messages that previously could not be decrypted due to missing the keys. 

`GroupKeyResponse`s must have `encryptionType=1 (RSA)` and encrypt the group keys delivered in the `content`. The group keys must be encrypted with the `rsaPublicKey` provided in the `GroupKeyRequest`. The `groupKeyId` field should be set to the value of `rsaPublicKey` to help the recipient identify the correct key pair.

The `content` encoded with `contentType=0` (JSON) is as follows:

```
[requestId, streamId, [groupKeys]]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `requestId`  | `string`  | The `requestId` of the `GroupKeyRequest`, identifying which request is being responded to.
 `streamId`   | `string`  | The stream for which a key is being delivered.
 `groupKeys`  | `array`   | Array of `[groupKeyId, encryptedGroupKeyHex]` pairs, containing the keys requested in the `GroupKeyRequest`. `encryptedGroupKeyHex` is a hex-encoded binary string, containing an RSA encrypted version of the group key.
 
Example of a `GroupKeyResponse` including the rest of the `Stream Protocol` fields:

```
[32, [...msgIdFields], [...msgRefFields], 29, 0, 1, "rsaPublicKey-from-the-request", "[\"requestId\", \"streamId\", [\"keyId\", \"encrypted-group-key\"]]", null, 2, "0x29c057786Fa..."]
```

Note that unlike `StreamMessage`s, here the whole `content` is not encrypted, only the group keys.

### GroupKeyAnnounce

When publishers want to rekey the stream, they send the `GroupKeyAnnounce` to each subscriber's key exchange stream. The message is sent RSA encrypted to each recipient's key exchange stream separately.

The recipient should handle this message by adding the contained key to their key storage.

The `content` encoded with `contentType=0` (JSON) is as follows:

```
[streamId, [groupKeys]]
```

 Field        | Type      | Description
 ------------ | --------- | -----------
 `streamId`   | `string`  | The stream for which a key is being delivered. 
  `groupKeys`  | `array`   | Array of [GroupKeys](#groupkey).
 
Examples of a `GroupKeyAnnounce` messages including the rest of the `Stream Protocol` fields:

```
[32, [...msgIdFields], [...msgRefFields], 30, 0, 1, "rsaPublicKey-of-recipient", "[\"streamId\", [\"newGroupKeyId\", \"RSA-encrypted-group-key-hex\"]]", null, 2, "0x29c057786Fa..."]
```

Note that unlike `StreamMessage`s, here the whole `content` is not encrypted, only the group keys.

### GroupKeyErrorResponse

In case there's a problem retrieving the keys requested via a `GroupKeyRequest`, this message gets sent by a publisher to the requestor's key exchange stream. Such error situations might include, for example:

- The requested key might not be found in the publisher's key store
- The requestor no longer has permissions to access the stream
- The request was incorrectly formed 

The error response contains the list of keys for which the exchange failed, and an explanation why.

`GroupKeyErrorResponse`s are always unencrypted (`encryptionType=0`). The `content` encoded with `contentType=0` (JSON) is as follows:

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
 
Example of a `GroupKeyErrorResponse` including the rest of the `Stream Protocol` fields:

```
[32, [...msgIdFields], [...msgRefFields], 31, 0, 0, null, "[\"requestId\", \"streamId\", \"ERROR_CODE\", \"Error message\", [\"groupKeyId1\"]]", null, 2, "0x29c057786Fa..."]
```

### MessageID

Uniquely identifies a Stream Protocol message.

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

Used inside a Stream Control message to identify the previous message on the same `msgChainId` (defined above). If the `MessageRef` is not provided by the publisher, then the stream will have weaker delivery guarantees, as subscribers will not be able to detect missing messages and request gapfills.

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


## Tracker Protocol

This document describes version `1` of the Tracker Control protocol.

All messages are transmitted as JSON arrays and have the following structure with certain fields common to all messages: `[version, type, requestId, ...typeSpecificFields]`. 

Field       | Type      | Description
----------- | --------- | -----------
`version`   | `number`  | The version of the Tracker Protocol
`type`      | `number`  | An integer identifying the message type (see the table below)
`requestId` |  `string` | Identifies a message. The `requestId` can be chosen freely for each request, as long as it's unique within the context of a particular connection. The same `requestId` value gets echoed back in response messages that result from the request.

The `...typeSpecificFields` means that there are additional fields depending on message type. The possible message `type`s are:

type | Description
----------- | -----------
1 | StatusMessage
2 | InstructionMessage
3 | StorageNodesRequest
4 | StorageNodesResponse


The individual types are described in the remainder of this section. We start by describing the requests and then the responses.

These messages are exchanged between trackers and nodes for peer discovery and topology formation.

### StatusMessage

A node reports its status to the tracker using this message.

```
[version, type, status]
```
Example:
```
[2, 14, {
    streams: [],
    started: 0,
    rtts: {}
}]
```

Field    | Type | Description
-------- | ---- | --------
`status` | `object` | state of node (details to be filled in later, subject to change)

### InstructionMessage

The tracker instructs nodes to connect to (and disconnect from) other nodes using this message.

```
[version, type, requestId, streamId, streamPartition, nodeAddresses, counter]
```
Example:
```
[2, 15, "request-id", "stream-id", 0, ["ws://address1", "ws://address2"], 10]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | stream id
`streamPartition` | `number` | stream partition
`nodeAddresses` | `array` | list of nodes the receiving node should be connected to
`counter` | `number` | incrementing counter value to keep track of latest instruction

### StorageNodesRequest

Sent by node to tracker to find a storage node for a stream.

```
[version, type, requestId, streamId, streamPartition]
```
Example:
```
[2, 16, "request-id", stream-id", 0]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | stream id
`streamPartition` | `number` | stream partition

### StorageNodesResponse

Response from tracker to node's `StorageNodesRequest` providing connection information of storage node.

```
[version, type, requestId, streamId, streamPartition, nodeAddresses]
```
Example:
```
[2, 17, "request-id", stream-id", 0, ["ws://storage-node-address-1", "ws://storage-node-address-2"]]
```

Field    | Type | Description
-------- | ---- | --------
`streamId` | `string` | stream id
`streamPartition` | `number` | stream partition
`nodeAddresses` | `array` | list of storage nodes associated with stream
