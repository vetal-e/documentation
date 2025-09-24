<a id="dev-cookbook-system-websockets-authentication-autorization"></a>

# Use Authentication and Authorization in WebSocket Connections

Despite the fact that WebSocket connections can be used to distribute messages to all site visitors independently of
their roles and permissions (e.g., to notify all visitors about new publications in the Company News section), in most
cases WebSocket messages are intended for a limited number of users that have appropriate permissions or interests to
publish or view messages in a particular topic.

To achieve this requirement, OroSyncBundle provides mechanisms for automatic client authentication.

All clients receive authentication tickets at the beginning of the connection. Before connecting, the client must
receive the connection ticket and pass it as the ticket query parameter in the connection URL.

For the frontend clients, the authentication ticket can be received by calling the POST request to the **oro_sync_ticket**
route. The response to this request is the JSON object with a ticket field containing a one-time authentication
ticket.

If the client is a backend client, the authentication ticket can be received by calling the **generateTicket** method of the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/SyncBundle/Authentication/Ticket/TicketProvider.php" target="_blank">oro_sync.authentication.ticket_provider</a> service.

A ticket can be of two types:

1. Representing an authenticated user.
2. Representing an anonymous client.

The anonymous client ticket could be used only from the backend to publish messages using the [WebSocket client](publish-to-topic.md#dev-cookbook-system-websockets-publish-to-topic) service.

The anonymous ticket is generated using a secret key in the application configuration and cannot be created without this key.

Authentication tickets have a limited lifetime of 300 seconds by default.

If the authentication is successful, the client is able to subscribe and send new messages to topics.

<!-- Frontend -->
