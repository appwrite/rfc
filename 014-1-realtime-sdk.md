# Realtime SDK - Solution Definition 

This document will describe the technical flow and behavior.

## Endpoint

The endpoint for the Realtime Server should be separate form the default endpoint. But will be set by default when the `setEndpoint` method is executed.

We will guess the Realtime Endpoint based on the HTTP Endpoint.

If it starts with `http://` => replace it with `ws://`.

If it starts with `https://` => replace it with `wss://`.

## Methods

### setEndpointRealtime

Will set the Endpoint for Realtime.

### subscribe

The method to subscribe to realtime updates should look like this:

`subscribe: (channels: string | string[], callback: (payload: object) => void) => () => void`

*@param* `channels`
Channel to subscribe - pass a single channel as a string or multiple with an array of strings.

Possible channels are:

- account
- collections
- collections.[ID]
- collections.[ID].documents
- documents
- documents.[ID]
- files
- files.[ID]

*@param* `callback` — Is called on every realtime update and passes the received payload as a single argument.

*@returns* — Returns a function to unsubscribes from the event/events.

## Connection Behavior 

At all times there should only be a single Connection established. Since the to be subscribed channels are passed by the URL - on every channel added or removed the current connection has to be terminated and a new connection has to be established.

This also means, that when a page loads and 20 channels are being subscribed to - we don't want the page to establish a connection 20 times. This should be handled via a debounce, that will wait if other connections are about to be established. For Javascript this is 1ms. This should be done everytime the subscribe method is being called.

## Disconnecting Behavior

When the connections is closed from the WebSocket, a new connection should be established after 1 second. To prevent unnecessary retry loops, the last message from the WebSocket server should be checked. If it is a JSON error message with the `code` value of 1008, no connection should be established again. 1008 means the connection was closed due to a *Policy violation* like wrong project ID or Rate Limits.
