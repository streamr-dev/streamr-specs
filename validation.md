# Validating SubscribeRequest and all ResendRequests

```
check that the sender has stream_subscribe permission to stream
```

# Validating PublishRequest

Validate the `StreamMessage` contained by the `PublishRequest`, as per below.

# Validating StreamMessage

The spec can be implemented by any code that needs to validate `StreamMessage`s they see, including:

1) SDKs, to provide automatic validation of incoming messages to apps using the network
2) nodes, which can validate messages to avoid propagating invalid messages to other nodes

## Validating normal payloads (contentType = 27)

```
if (message is unsigned) {
    check that stream.requireSignedData == false
} else {
    check that the signature is correct
    check that the message publisher has stream_publish permission to the stream
}
```

## Validating group key requests (contentType = 28)

```
let S be the stream for which the key request is

check that the message was received on a key exchange stream (see below)
check that the signature is correct
check that the publisher has stream_subscribe permission to S
```

## Validating group key responses (contentType = 29) and resets (contentType = 30)

```
let S be the stream for which the key response/reset is

check that the message was received on a key exchange stream (see below)
check that the signature is correct
check that the publisher has stream_publish permission to S
```

## Key exchange streams

Key exchange streams have a special id of the form

```
SYSTEM/keyexchange/{address}
```

Where `{address}` is as follows:

- For group key requests, the `{address}` is the `publisherId` (Ethereum address) of the message that the requestor needs to decrypt.
- For group key responses, the `{address}` is the address that signed the group key request for which this is the response.

Key exchange streams are ethereal and don't explicitly exist as streams. Rather they exist as implicit streams with one partition (`0`) and no explicit metadata or permissions attached to them.

Publishers publishing encrypted messages should subscribe to the key exchange stream corresponding to their Ethereum address in order to receive group key requests.
