# Table of Contents

- [Control Layer](#control-layer)
- [Stream Layer](#stream-layer)
- [Key exchange streams](#key-exchange-streams)

# Control Layer

## PublishRequest, BroadcastMessage, and UnicastMessage

Validated by validating the wrapped [Stream Layer](#stream-layer) message.

## SubscribeRequest and ResendRequests

```
if (stream is a key exchange stream) {
    check that the sender's address equals the address of the requested key exchange stream
    check that streamPartition == 0
} else {
    check that the sender has stream_subscribe permission to stream
}
```

Note: At the moment, with no handshake procedure in place yet to establish the address of the connecting client in all cases, it's not possible to implement the above exactly for [key exchange streams](#keyexchangestreams). As a workaround, we can safely let anyone subscribe to key exchange streams:

```
if (stream is a key exchange stream) {
    // TODO: enable later when clients do handshakes
    // check that the sender's address equals the address of the requested key exchange stream
    check that streamPartition == 0
} else {
    check that the sender has stream_subscribe permission to stream
}
```
# Stream Layer

## StreamMessage (messageType = 27)

```
if (message is unsigned) {
    check that stream.requireSignedData == false
} else {
    check that the signature is correct
    check that the message publisher has stream_publish permission to the stream
}
```

Note: support for unsigned messages will be dropped later.

## GroupKeyRequest (messageType = 28)

```
let S be the stream for which the key request is

check that the message was received on a key exchange stream (see below)
check that the signature is correct
check that the publisher has stream_subscribe permission to S
```

## GroupKeyResponse (messageType = 29) and GroupKeyReset (messageType = 30)

```
let S be the stream for which the key response/reset is

check that the message was received on a key exchange stream (see below)
check that the signature is correct
check that the publisher has stream_publish permission to S
```

## GroupKeyRotate (messageType = 30)

```
check that the message is signed and encrypted
then validate the message as if it was a StreamMessage
```

# Key exchange streams

Key exchange streams have a special id of the form

```
SYSTEM/keyexchange/{address}
```

Where `{address}` is as follows:

- For group key requests, the `{address}` is the `publisherId` (Ethereum address) of the message that the requestor needs to decrypt.
- For group key responses, the `{address}` is the address that signed the group key request for which this is the response.

Key exchange streams are ethereal and don't explicitly exist as streams. Rather they exist as implicit streams with one partition (`0`) and no explicit metadata or permissions attached to them.

Publishers publishing encrypted messages should subscribe to the key exchange stream corresponding to their Ethereum address in order to receive group key requests.
