<a id="dev-guide-system-websockets"></a>

<a id="dev-guide-system-websockets-architecture"></a>

# WebSocket Notifications

**WebSockets** is a full-duplex communication protocol for real-time messaging between a server and clients through persistent connections.

The main aim of WebSockets is to provide real-time notifications for clients about server events or changes without a need for the client to repeat requests to the server for new information. For example:

* When someone changed the document, which another user was in the progress of editing. In this case, a notification informing that someone is working with this document, or that the document has been modified is immensely helpful.
* Real-time charts of stock prices or currency exchange rates on financial portals. The accuracy and timeliness of this type of data are crucial for visitors of the portal, and manual refreshment of the page can be very exhausting.
* Real-time instant messaging on the website. Users must receive messages without refreshing the chat page, and so on.

In Oro applications, WebSocket communications are built using <a href="https://wamp-proto.org/" target="_blank">Web Application Message Protocol (WAMP)</a>, a WebSocket subprotocol aimed at organizing the communication between program components in the applications with a loosely coupled architecture.

The main two parts of WAMP protocol are <a href="https://en.wikipedia.org/wiki/Remote_procedure_call" target="_blank">Remote Procedure Call</a> (RPC) mechanism and <a href="https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern" target="_blank">PubSub</a> messaging pattern.

**RPC** mechanism allows calling a function from a different code remotely via a WebSocket.

**PubSub** messaging pattern implies that when messages are published to topics by publishers (or, in other words, “channels”), the broker distributes them to clients that are subscribed to these topics.

Therefore, the **WAMP** protocol implies that there is a **WebSocket server** that plays the role of message broker; there are ways for the
application components to **register topics** for messages, **publish messages** to topics, and **subscribe to topic** messages.

In Oro applications, all WebSocket-related functionality is provided by <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/SyncBundle/" target="_blank">OroSyncBundle</a>. As OroSyncBundle is
part of OroPlatform, which is the base for all Oro applications, the WebSocket functionality exists in all Oro
applications.

#### NOTE
WebSocket functionality exists only in the Oro application admin UI which guarantees authentication of all clients who subscribe to the topic messages.

## Getting Started

You need to [Setup and Configure](configuration/index.md#dev-guide-system-websockets-setup-configuration) websocket functionality before you can start using it in Oro applications.

Out-of-the-box, OroSyncBundle uses WebSocket connection for two purposes:

* [Content outdated notifications](recipes/content-outdating-notifications.md#dev-cookbook-system-websockets-content-outdating-notifications) — To provide flash notifications for the user informing about outdated content, if several users try to edit the same entity record simultaneously.
* [Maintenance mode notifications](recipes/maintenance-mode.md#dev-cookbook-system-websockets-maintenance-mode) — To send flash notifications to all application site visitors once a developer turns on the system maintenance mode by a console’s CLI tool.

To start using websocket messages for your custom functionality, refer to the following articles:

* [Create Your Own Topic for Publishing and Subscribing](recipes/create-topic-and-handler.md#dev-cookbook-system-websockets-create-topic-and-handler)
* [Publish Messages to Existing Topics](recipes/publish-to-topic.md#dev-cookbook-system-websockets-publish-to-topic)

<!-- Frontend -->
