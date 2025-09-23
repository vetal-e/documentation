<a id="op-structure-mq-divide-single-to-separate"></a>

# Divide Queue to Separate Queues

There are a couple of cases when you need to move messages from one queue to another using **routing key**, for example:

* When configuring **multiple queue**. If all your messages are stored in a single queue (like `oro.default`), you may want to configure more than one queue and split all your messages.
* When setting up **backup restore**. You may need to exclude messages after the backup (like email notifications, 3rd party application updates, etc.)

Use the <a href="https://www.rabbitmq.com/shovel.html" target="_blank">Shovel plugin</a> to divide your single queue into separate queues.

## Procedure

1. <a href="https://www.rabbitmq.com/shovel.html#getting-started" target="_blank">Install Shovel Plugin</a>
2. <a href="https://www.rabbitmq.com/shovel.html#management-status" target="_blank">Install Shovel Management Plugin</a>
3. [Configure Temporary Queue]()
4. [Relocate Messages to Temporary Queue]()
5. [Configure Alternate Exchange]()
6. [Configure Dividing Exchange]()
7. [Create Additional Queues]()
8. [Configure Bindings]()
9. [Relocate Messages to Separate Queues]()
10. [Clear Temporary Exchanges and Queues]()

## Example

The example below illustrates how to consume messages from the oro.default queue on the local broker and push them to separate queues, oro.email and oro.notification. All other messages are returned to the oro.default queue.

Use the commands provided by the [RabbitMQ management plugin](rabbitmq-command-lines.md#op-structure-mq-rabbit-command-lines):

* declare queue
* declare exchange
* declare binding

### Configure Temporary Queue

Create the oro.temporary queue on your host with durable=true and x-max-priority=4.

```none
rabbitmqadmin declare queue --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary" durable=true arguments='{"x-max-priority": 4}'
```

### Relocate Messages to Temporary Queue

Declare the oro.temporary exchange with the fanout type to be able to move messages to oro.temporary with the original **routing_key**.

```none
rabbitmqadmin declare exchange --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary" type="fanout" durable=true
```

Bind the oro.temporary exchange with the oro.temporary queue. All message from this exchange will be transferred to the oro.default queue.

```none
rabbitmqadmin declare binding --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
source="oro.temporary" destination="oro.temporary" destination_type=queue
```

Create Dynamic Shovel to relocate all message from the oro.default to the oro.temporary queue.

```none
rabbitmqctl set_parameter shovel temporary-queue \
'{"src-protocol": "amqp091", "src-uri": "amqp://localhost:5672", "src-queue": "oro.default", '\
'"dest-protocol": "amqp091", "dest-uri": "amqp://localhost:5672", "dest-exchange": "oro.temporary", '\
'"src-delete-after": "queue-length"}'
```

Check if all messages are relocated.

```none
rabbitmqctl shovel_status | grep "temporary-queue"
```

Shovel will be removed automatically if you specify the src-delete-after=queue-length option.

Remove the oro.temporary exchange.

```none
rabbitmqadmin delete exchange --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary"
```

### Configure Alternate Exchange

Declare the oro.temporary.alternate exchange with the fanout type to be able to move messages to the oro.default queue that does not contain the **routing_key** described in the binding mask.

```none
rabbitmqadmin declare exchange --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary.alternate" type="fanout" durable=true
```

### Configure Dividing Exchange

Declare the oro.temporary.divide exchange with the topic type to be able to move messages to separate queues by **routing_key**. Specify the alternate-exchange=oro.temporary.alternate option that marks the oro.temporary.alternate exchange as a **alternate exchange** for the oro.temporary.divide exchange.

```none
rabbitmqadmin declare exchange --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary.divide" type="topic" durable=true \
arguments='{"alternate-exchange": "oro.temporary.alternate"}'
```

### Create Additional Queues

Declare the oro.email queue with durable=true and x-max-priority=4.

```none
rabbitmqadmin declare queue --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name=oro.email durable=true arguments='{"x-max-priority": 4}'
```

Declare the oro.notification queue with durable=true and x-max-priority=4.

```none
rabbitmqadmin declare queue --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name=oro.notification durable=true arguments='{"x-max-priority": 4}'
```

### Configure Bindings

Bind the oro.temporary.alternate exchange with the oro.default queue. All messages from this exchange will be transferred to the oro.default queue.

```none
rabbitmqadmin declare binding --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
source="oro.temporary.alternate" destination="oro.default" destination_type=queue
```

Bind the oro.temporary.divide exchange with the oro.email queue by routing_key=oro.email.#

```none
rabbitmqadmin declare binding --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
source="oro.temporary.divide" destination="oro.email" destination_type=queue \
routing_key="oro.email.#"
```

Bind the oro.temporary.divide exchange with the oro.notification queue by routing_key=oro.notification.#

```none
rabbitmqadmin declare binding --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
source="oro.temporary.divide" destination="oro.notification" destination_type=queue \
routing_key="oro.notification.#"
```

### Relocate Messages to Separate Queues

```none
rabbitmqctl set_parameter shovel divide-queue \
'{"src-protocol": "amqp091", "src-uri": "amqp://localhost:5672", "src-queue": "oro.temporary", '\
'"dest-protocol": "amqp091", "dest-uri": "amqp://localhost:5672", "dest-exchange": "oro.temporary.divide", '\
'"ack-mode": "on-publish", "src-delete-after": "queue-length"}'
```

Check if all messages are relocated.

```none
rabbitmqctl shovel_status | grep "divide-queue"
```

### Clear Temporary Exchanges and Queues

Remove the oro.temporary.alternate exchange.

```none
rabbitmqadmin delete exchange --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary.alternate"
```

Remove the oro.temporary.divide exchange.

```none
rabbitmqadmin delete exchange --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary.divide"
```

Remove the oro.temporary queue.

```none
rabbitmqadmin delete queue --host=$HOST --user=$USER --password=$PASSWORD --vhost=$VHOST \
name="oro.temporary"
```

For more information, see the following external resources:

* <a href="https://www.rabbitmq.com/shovel.html" target="_blank">RabbitMQ Shovel Plugin</a>
* <a href="https://www.rabbitmq.com/shovel-dynamic.html" target="_blank">RabbitMQ Configuring Dynamic Shovels</a>

<!-- Frontend -->
