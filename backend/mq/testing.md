# Testing

<a id="dev-cookbook-system-mq-testing"></a>

## Create and Run Unit and Functional Tests to Test the Message Queue

To test that a message was sent in unit and functional tests, you can use the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MessageQueueBundle/Test/Assert/AbstractMessageQueueAssertTrait.php" target="_blank">AbstractMessageQueueAssertTrait</a> trait.
There are two implementations of this trait, one for unit tests and the other one for functional tests:

* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MessageQueueBundle/Test/Unit/MessageQueueExtension.php" target="_blank">Oro\\Bundle\\MessageQueueBundle\\Test\\Unit\\MessageQueueExtension</a> is for unit tests
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MessageQueueBundle/Test/Functional/MessageQueueExtension.php" target="_blank">Oro\\Bundle\\MessageQueueBundle\\Test\\Functional\\MessageQueueExtension</a> is for functional tests

If you need a custom logic to manage sent messages, you can use the
<a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MessageQueueBundle/Test/Unit/MessageQueueAssertTrait.php" target="_blank">Oro\\Bundle\\MessageQueueBundle\\Test\\Unit\\MessageQueueAssertTrait</a> or
<a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/MessageQueueBundle/Test/Functional/MessageProcessTrait.php" target="_blank">Oro\\Bundle\\MessageQueueBundle\\Test\\Functional\\MessageQueueAssertTrait</a> traits.

The following example shows how to test whether a message was sent.

```php
namespace Acme\Bundle\DemoBundle\Tests\Functional;

use Oro\Bundle\MessageQueueBundle\Test\Functional\MessageQueueExtension;
use Oro\Bundle\TestFrameworkBundle\Test\WebTestCase;

class SomeTest extends WebTestCase
{
    use MessageQueueExtension;

    public function testSingleMessage()
    {
        // assert that a message was sent to a topic
        self::assertMessageSent('aFooTopic', 'Something has happened');

        // assert that at least one message was sent to a topic
        // can be used if a message is not matter
        self::assertMessageSent('aFooTopic');
    }

    public function testSeveralMessages()
    {
        // assert that exactly given messages were sent to a topic
        self::assertMessagesSent(
            'aFooTopic',
            [
                'Something has happened',
                'Something else has happened',
            ]
        );
        // assert that the exactly given number of messages were sent to a topic
        // can be used if messages are not matter
        self::assertMessagesCount('aFooTopic', 2);
        // also assertCountMessages alias can be used to do the same assertion
        self::assertCountMessages('aFooTopic');
    }

    public function testNoMessages()
    {
        // assert that no any message was sent to a topic
        self::assertMessagesEmpty('aFooTopic');
        // also assertEmptyMessages alias can be used to do the same assertion
        self::assertEmptyMessages('aFooTopic');
    }

    public function testAllMessages()
    {
        // assert that exactly given messages were sent
        // NOTE: use this assertion with caution because it is possible
        // that messages not related to a testing functionality were sent as well
        self::assertAllMessagesSent(
            [
                ['topic' => 'aFooTopic', 'message' => 'Something has happened'],
                ['topic' => 'aFooTopic', 'message' => 'Something else has happened'],
            ]
        );
    }
}
```

In unit tests, you are usually need to pass the message producer to a service you are testing. To fetch the correct instance of the
message producer in the unit tests, use self::getMessageProducer().

For example:

```php
namespace Acme\Bundle\DemoBundle\Tests\Unit;

use Acme\Bundle\DemoBundle\SomeClass;
use Oro\Bundle\MessageQueueBundle\Test\Unit\MessageQueueExtension;

class SomeTest extends \PHPUnit\Framework\TestCase
{
    use MessageQueueExtension;

    public function testSingleMessage()
    {
        $instance = new SomeClass(self::getMessageProducer());

        $instance->doSomethind();

        self::assertMessageSent('aFooTopic', 'Something has happened');
    }
}
```

<!-- Frontend -->
