<a id="dev-guide-mq-filtering-messages"></a>

# Filtering Messages in the Message Producer

The filtering of messages is introduced to be able to filter messages before sending them to the message queue.
Together with the [buffering of messages](buffering-messages.md#dev-guide-mq-buffering-messages), it allows removing duplicated messages,
combining several messages in one message or even removing messages on the client side and, as a result,
sending an optimized set of messages to the message queue.

To create a new message filter, you need to create a class that implements <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MessageQueueBundle/Client/MessageFilterInterface.php" target="_blank">MessageFilterInterface</a>
and register it in the dependency injection container with the `oro_message_queue.message_filter` tag.
This tag has the topic attribute that is used to specify a topic to which a filter should be applied.
When the topic is not specified, the filter is applied to messages from all topics.
The priority attribute of the tag can be used to specify the order in which the filter should be applied.
The higher the number, the earlier the filter is applied.

An example of a message filter that removes duplicated messages:

```php
namespace Acme\Bundle\DemoBundle\Async;

use Oro\Bundle\MessageQueueBundle\Client\MessageBuffer;
use Oro\Bundle\MessageQueueBundle\Client\MessageFilterInterface;

class MyMessageFilter implements MessageFilterInterface
{
    private string $topic;

    /**
     * @param string $topic
     */
    public function __construct(string $topic)
    {
        $this->topic = $topic;
    }

    #[\Override]
    public function apply(MessageBuffer $buffer): void
    {
        if (!$buffer->hasMessagesForTopic($this->topic)) {
            return;
        }

        $processedMessages = [];
        $messages = $buffer->getMessagesForTopic($this->topic);
        foreach ($messages as $messageId => $message) {
            $messageKey = (string)$message['id'];
            if (isset($processedMessages[$messageKey])) {
                $buffer->removeMessage($messageId);
            } else {
                $processedMessages[$messageKey] = true;
            }
        }
    }
}
```

```yaml
services:
    acme_demo.my_message_filter:
        class: Acme\Bundle\DemoBundle\Async\MyMessageFilter
        arguments:
            - !php/const Acme\Bundle\DemoBundle\Async\Topic\MySampleTopic::NAME
        tags:
            - { name: oro_message_queue.message_filter, topic: !php/const Acme\Bundle\DemoBundle\Async\Topic\MySampleTopic::NAME }
```

#### NOTE
- The filtering is not executed after filters made changes in the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MessageQueueBundle/Client/MessageBuffer.php" target="_blank">message buffer</a>.
- All messages represented by <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Component/MessageQueue/Client/MessageBuilderInterface.php" target="_blank">message builders</a> are resolved before the filtering is executed.

<!-- Frontend -->
