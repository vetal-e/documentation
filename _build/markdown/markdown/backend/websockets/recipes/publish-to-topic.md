<a id="dev-cookbook-system-websockets-publish-to-topic"></a>

# Publish Messages to Existing Topics

## Publish Messages from Backend

To publish and read messages from WebSocket connections topics on the backend side of your Oro application, OroSyncBundle provides a WebSocket  **oro_sync.websocket_client** client which is based on the Gos WebSocketClient component *Gos\\Component\\WebSocketClient\\Wamp\\Client*.

WebSocket client uses [authentication tickets mechanism](authentication-autorization.md#dev-cookbook-system-websockets-authentication-autorization), so you should not worry about authentication on the backend side.

#### NOTE
WebSocket client oro_sync.websocket_client uses the **anonymous** authentication tickets, so when you connect to WebSocket server, it treats you as an anonymous user.

You can publish messages to channels using the publish() method of the **oro_sync.websocket_client**, e.g.,:

```php
$websocketClient = $this->get('oro_sync.websocket_client');
$websocketClient->publish('oro/custom-channel', ['foo' => 'bar']);
```

It is strongly recommended that you use the **oro_sync.client.connection_checker** connection checker before trying to publish or connect to websocket server, e.g.,:

```php
$websocketConnectionChecker = $this->get('oro_sync.client.connection_checker');
if ($websocketConnectionChecker->checkConnection()) {
    $websocketClient = $this->get('oro_sync.websocket_client');
    $websocketClient->publish('oro/custom-channel', ['foo' => 'bar']);
}
```

## Publish Messages from Frontend

So far, there is no options to publish messages to websocket topics from the frontend side.

## List All Declared Topics

All declared topics for WebSocket connections in your application (that you can subscribe and publish messages to) are declared in the *websocket_routing.yml* files in the *Resources/config/oro/* folders in application bundles. You can search all these files and look into them to find all declared topics.

#### NOTE
For more details on how to declare topics, see the [Create Your Topic and Handler for Publishing and Subscribing](create-topic-and-handler.md#dev-cookbook-system-websockets-create-topic-and-handler) topic.
