# Message Validation

This is a specification for how incoming `StreamMessage`s should be validated. The spec can be implemented by any code that needs to validate messages they see, including:

1) client libraries, to provide automatic validation of incoming messages to apps using the network
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

check that the message was received on key exchange stream
check that the signature is correct
check that the publisher has stream_subscribe permission to S
```

## Validating group key responses (contentType = 29) and resets (contentType = 30)

```
let S be the stream for which the key response/reset is

check that the message was received on an key exchange stream
check that the signature is correct
check that the publisher has stream_publish permission to S
```
