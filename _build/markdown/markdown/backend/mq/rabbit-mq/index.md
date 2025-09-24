<a id="op-structure-mq-rabbitmq-intro"></a>

<a id="op-structure-mq-rabbitmq"></a>

# RabbitMQ (Enterprise Edition Only)

## AmqpMessageQueue Component

The component incorporates message queue in your application via
different transports. It contains several layers.

The lowest layer is called Transport and it provides an abstraction of
transport protocol. The Consumption layer provides the tools to consume
messages, such as the cli command, signal handling, logging, extensions. It
works on top of the transport layer.

The Client layer provides the ability to start
`producing\consuming` messages with as little configuration as possible.

## Installation

You need to have RabbitMQ **version 3.7.21**  and above installed to use the AMQP
transport. To install the RabbitMQ you should follow the <a href="https://www.rabbitmq.com/download.html" target="_blank">download and installation manual</a>.

After the installation, please check that you have all the required plugins
installed and enabled.

## Minimum Permissions

#### NOTE
You might want to read more on <a href="https://www.rabbitmq.com/access-control.html" target="_blank">access control</a>.

Your credentials must meet the next minimum requirements:

- You have access to the requested rabbitmq’s virtual host (`/` by
  default).
- You need to have the next permissions: `configure`, `write`,
  `read`. It could be a default value `.*` or a stricter
  `oro\..*`.

## RabbitMQ Plugins

### Required plugins

| Plugin name                                  | Version   | Appointment                                                                                                                                                                                                           |
|----------------------------------------------|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| rabbitmq_del<br/>ayed_message<br/>\_exchange | 3.8.0     | A plugin that adds delayed-messaging (or<br/>scheduled-messaging) to RabbitMQ.<br/><a href="https://github.com/rabbitmq/rabbitmq-delayed-message-exchange" target="_blank">Read more on Delayed Message Exchange</a>. |

The plugin `rabbitmq_delayed_message_exchange` is necessary
for the proper work but it is not installed by default, so you need to
download, install and enable it.

To download it, use the following command:

```none
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.8.0/rabbitmq_delayed_message_exchange-3.8.0.ez -P $RABBITMQ_HOME/plugins
```

To enable it, use the following command:

```none
rabbitmq-plugins enable --offline rabbitmq_delayed_message_exchange
```

### Recommended plugins

| Plugin name         | Version   | Appointment                                                                                                                                                                                      |
|---------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| rabbitmq_management | 3.8.\*    | Provides an HTTP-based API<br/>for management and<br/>monitoring of your RabbitMQ<br/>server.<br/><a href="https://www.rabbitmq.com/management.html" target="_blank">Read more on Management</a> |

### Plugins management

To enable plugins, use the `rabbitmq-plugins` tool:
`rabbitmq-plugins enable plugin-name`

And to disable plugins again, use:
`rabbitmq-plugins disable plugin-name`

To see the list of enabled plugins, use:
`rabbitmq-plugins list  -e`

You will see something like:

```none
[E*] rabbitmq_delayed_message_exchange 3.8.0
[E*] rabbitmq_management               3.8.2
[e*] rabbitmq_management_agent         3.8.2
[e*] rabbitmq_web_dispatch             3.8.2
```

The sign `[E*]` means that the plugin was explicitly enabled, i.e.
somebody enabled it manually. The sign `[e*]` means the plugin was
implicitly enabled, i.e. enabled automatically as it was required for
a different enabled plugin.

* <a href="https://www.rabbitmq.com/community-plugins.html" target="_blank">More about RabbitMQ plugins</a>
* <a href="https://www.rabbitmq.com/plugins.html" target="_blank">More about RabbitMQ plugins</a>

## Queues

If you use only this component, you are free to create any queues you
want and as many as you need. If you use the Client abstraction
with this transport, the next queues will be created: `oro.default` and
`oro.default.delayed`. The first keeps all sent messages, and the
seconds keeps broken message that have to be delayed and redelivered
later. You can still have more queues by explicitly configuring the message
processor `destinationName` option.

## Default Queue Presets

### Exchanges

| Name                | Type              | Features                              |
|---------------------|-------------------|---------------------------------------|
| oro.default         | fanout            | durable: true                         |
| oro.default.delayed | x-delayed-message | durable: true; x-delayed-type: fanout |

### Queues

| Name        | Features                         |
|-------------|----------------------------------|
| oro.default | durable: true; x-max-priority: 4 |

## Delaying Messages

In order to use delayed message with RabbitMQ broker, you have to install
its plugin. Read more on <a href="https://www.rabbitmq.com/blog/2015/04/16/scheduling-messages-with-rabbitmq/" target="_blank">scheduling messages on RabbitMQ website</a>.

## Usage

Usage is similar to one described in the message queue component. Here,
we will show you how to get amqp connection. We are assuming the
RabbitMQ is used as a broker with minimum configuration.

```php
use Oro\Component\AmqpMessageQueue\Transport\Amqp\AmqpConnection;

$connection = AmqpConnection::createFromConfig([
    'host' => '127.0.0.1',
    'port' => 5672,
    'user' =>  'guest',
    'password' => 'guest',
    'vhost' => '/',
]);
```

In order to use the component with the symfony application, you first have to
register the amqp transport factory (). And tell the message queue
bundle to use it.

```php
namespace Oro\Bundle\AmqpMessageQueueBundle;

use Oro\Bundle\MessageQueueBundle\DependencyInjection\OroMessageQueueExtension;
use Oro\Component\AmqpMessageQueue\DependencyInjection\AmqpTransportFactory;
use Symfony\Component\DependencyInjection\ContainerBuilder;
use Symfony\Component\HttpKernel\Bundle\Bundle;

class AcmeCoreBundle extends Bundle
{
    #[\Override]
    public function build(ContainerBuilder $container): void
    {
        parent::build($container);

        /** @var OroMessageQueueExtension $extension */
        $extension = $container->getExtension('oro_message_queue');
        $extension->addTransportFactory(new AmqpTransportFactory());
    }
}
```

#### NOTE
You can use AmqpMessageQueueBundle to register the factory automatically

You can use the `ORO_MQ_DSN` environment variable:

```bash
ORO_MQ_DSN=amqp://guest:guest@localhost:5672/%2Fmaster
```

When configuring a virtual host (vhost), ensure that the vhost is URL encoded.
If no vhost is provided, the default value of `/` is used.
For example, if the vhost is `/master`, the corresponding url encoded vhost value is `%2Fmaster`, and if the vhost is `master`,the url encoded value is `master`.

## RabbitMQ Useful Hints

- You can see the RabbitMQ default web interface here, if the
  `rabbitmq_management` plugin is enabled:
  `http://localhost:15672/`. <a href="https://www.rabbitmq.com/management.html" target="_blank">Read more on Management</a>.
- You can temporary stop RabbitMQ by running the command
  `rabbitmqctl stop_app`. The command will stop the RabbitMQ
  application, leaving the Erlang node running. You can resume it with
  the command `rabbitmqctl start_app`. <a href="https://www.rabbitmq.com/rabbitmqctl.8.html" target="_blank">Read more on rabbitmqctl(8)</a>.

<!-- Frontend -->
