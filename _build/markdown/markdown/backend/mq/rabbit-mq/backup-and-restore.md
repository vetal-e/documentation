<a id="op-structure-mq-backup-restore"></a>

# Backup and Restore

This section provides recommendations on how to back up and restore the message queue when working with Oro applications. For more information on how to back up and restore RabbitMQ, see the related <a href="https://www.rabbitmq.com/backup.html" target="_blank">documentation on RabbitMQ website</a>.

## Backup

RabbitMQ backups are JSON representation of your broker’s metadata which includes users, vhosts, queues, exchanges, and bindings.

### Backup Procedure

1. Stop all consumers
2. Export definitions
3. Back up messages
4. Start all consumers

#### NOTE
Maintenance mode is **not required** for creating RabbitMQ backup.

### Exporting Definitions

Use the `export` command provided by the [RabbitMQ management plugin](rabbitmq-command-lines.md#op-structure-mq-rabbit-command-lines) to back up definitions.

```none
rabbitmqadmin export /path/to/your/backup/directory/definitions.backup
```

### Backing Up Messages

**Stop all consumers** to back up messages. Messages’ data is stored in the <a href="https://www.rabbitmq.com/relocate.html" target="_blank">node's data directory</a>. Here, the nodes are collected in the `msg_stores/vhosts` subdirectory which has other directories created per vhost.

The only way to back up messages is to **copy \`\`msg_stores\`\` messages data directory**.

```none
# check your node directory
rabbitmqctl eval 'rabbit_mnesia:dir().'

# copy messages
cp -a /var/lib/rabbitmq/mnesia/your\@node/msg_stores/ /path/to/your/backup/directory/msg_stores
```

## Restore

For messages to be restored, the broker should have all the definitions already in place. Message data for unknown vhosts and queues will not be loaded and can be deleted by the node.

### Restore Procedure

1. Stop all consumers
2. Enable maintenance mode
3. Import definitions
4. Restore messages
5. Exclude messages
6. Stop maintenance mode
7. Start all consumers

### Import Definitions

Use the `import` command provided by the [RabbitMQ management plugin](rabbitmq-command-lines.md#op-structure-mq-rabbit-command-lines) to restore definitions.

```none
rabbitmqadmin import /path/to/your/backup/directory/definitions.backup
```

### Restore Messages

Copy `msg_stores` messages data directory to the <a href="https://www.rabbitmq.com/relocate.html" target="_blank">node's data directory</a>.

```none
# stop RabbitMQ node
rabbitmqctl stop_app

# copy messages
cp -a /path/to/your/backup/directory/msg_stores/* /var/lib/rabbitmq/mnesia/your\@node/msg_stores/

# start RabbitMQ node
rabbitmqctl start_app
```

#### NOTE
If your **backup node name is different**, you can change it as described in the <a href="https://www.rabbitmq.com/backup.html" target="_blank">documentation on RabbitMQ website</a>.

### Exclude Messages (emails notifications, 3rd party application updates, etc.)

**To prevent reprocessing messages** that contain email notifications, 3rd party application update, etc., filter them by **routing key** and divide a single queue into separate queues, as described in the [Divide Single Queue to Separate Queues](divide-single-queue-to-separate-queues.md#op-structure-mq-divide-single-to-separate) topic. After that, you can easily remove separate queues with excluded messages.

For more information, see the following external resources:

* <a href="https://www.rabbitmq.com/backup.html" target="_blank">RabbitMQ Backup and Restore</a>
* <a href="https://www.rabbitmq.com/relocate.html" target="_blank">RabbitMQ File and Directory Locations</a>

<!-- Frontend -->
