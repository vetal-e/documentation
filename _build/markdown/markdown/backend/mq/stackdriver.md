<a id="dev-guide-mq-stackdriver"></a>

# Writing Logs to Stackdriver

<a href="https://cloud.google.com/stackdriver" target="_blank">Google Stackdriver</a> logging allows you to store, search, analyze, monitor, and alert on log data.

Logs can be streamed to the Stackdriver with <a href="https://cloud.google.com/logging/docs/agent/" target="_blank">Logging Agent</a> or manually using <a href="https://github.com/GoogleCloudPlatform/google-cloud-php" target="_blank">GCloud SDK</a>.

A simple example of how you can stream your message queue logs to the Stackdriver with custom <a href="https://symfony.com/doc/current/logging.html#handlers-that-modify-log-entries" target="_blank">Monolog Handler</a> is described below.

Add Google Cloud SDK package:

```bash
COMPOSER=dev.json composer require "google/cloud":"~0.70"
```

Create **StackdriverHandler** that uses GCloud SDK:

#### NOTE
src/Acme/Bundle/DemoBundle/Log/Handler/StackdriverHandler.php
```php
 namespace Acme\Bundle\DemoBundle\Log\Handler;

 use Google\Cloud\Logging\LoggingClient;
 use Monolog\Handler\PsrHandler;
 use Monolog\Logger;

 class StackdriverHandler extends PsrHandler
 {
     /**
      * StackdriverHandler constructor.
      */
     public function __construct()
     {
         $client = new LoggingClient([
             'projectId' => 'Your Project Id',
         ]);

         $logger = $client->psrLogger('oro_message_queue');

         // In current example logs with level Logger::NOTICE and higher will be streamed to the Stackdriver
         parent::__construct($logger, Logger::NOTICE);
     }

     #[\Override]
     public function handle(array $record)
     {
         $record['context']['extra'] = $record['extra'];

         return parent::handle($record);
     }
 }
```

Define the service in the DI:

```yaml
 services:
     acme_demo.log.handler.stackdriver:
         class: Acme\Bundle\DemoBundle\Log\Handler\StackdriverHandler
```

Configure Monolog handler to listen only to message queue logs:

```yaml
 monolog:
     handlers:
         message_queue.consumer.stackdriver:
             type: service
             id: acme_demo.log.handler.stackdriver
             channels: ['!mq_job_transitive']
```

Check the *Google Cloud Platform > Stackdriver > Logging > Logs*, now you can configure filters and monitor all logs streamed to the Stackdriver.

![Stackdriver Filter](img/backend/mq/stackdriver_filter.png)

## Logs-Based Metrics

Logs-based metrics is a useful tool to check whats is going on with message consuming. More information is available in the <a href="https://cloud.google.com/logging/docs/logs-based-metrics/" target="_blank">Overview of logs-based metrics</a> topic.

Few examples of logs-based metrics are listed below:

1. The total number of statuses received.
   * Check <a href="https://cloud.google.com/logging/docs/logs-based-metrics/counter-metrics" target="_blank">How to Create Counter Metric</a>.
   * In the viewer panel, create a filter `jsonPayload.status=("REJECT" OR "REQUEUE" OR "ACK")`; it shows only the message processed log entries
   * In the **Metric Editor** panel, set the following fields:

   | Field                                                     | Value                                                                                                                                                                                                                                             |
   |-----------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   | **Name**<br/>**Add Label**<br/>**Add Label**<br/>**Type** | status_count<br/>**Name**: topic, **Label type**: String, **Field name**: jsonPayload.extra.message_properties.”oro.message_queue.client.topic_name”<br/>**Name**: status, **Label type**: String, **Field name**: jsonPayload.status<br/>Counter |

   The current metrics shows total number of all statuses grouped by **Topic Name** with **Status**.
   ![Total number of statuses metric](img/backend/mq/stackdriver_total_number.png)
2. Memory peak of received messages.
   * Check <a href="https://cloud.google.com/logging/docs/logs-based-metrics/distribution-metrics" target="_blank">How to Create Distribution Metric</a>.
   * In the viewer panel, create a filter `jsonPayload.status=("REJECT" OR "REQUEUE" OR "ACK")` it shows only the message processed log entries
   * In the **Metric Editor** panel, set the following fields:

   | Field                                                                                                                                                                                                                | Value                                                                                                                                                                                                                                                                                                                                                                                             |
   |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   | **Name**<br/>**Add Label**<br/>**Add Label**<br/>**Type**<br/>**Field name**<br/>**Extraction regular expression**<br/>**Histogram buckets Type**<br/>**Start value**<br/>**Number of buckets**<br/>**Bucket width** | peak_memory<br/>**Name**: topic, **Label type**: String, **Field name**: jsonPayload.extra.message_properties.”oro.message_queue.client.topic_name”<br/>**Name**: status, **Label type**: String, **Field name**: jsonPayload.status<br/>Distribution<br/>jsonPayload.extra.peak_memory<br/>([0-9.]+) MB<br/>Linear  *(setup current and below values by yours preferences)*<br/>40<br/>200<br/>5 |

   The current metrics shows Peak of Memory for each message grouped by **Topic Name** with **Status**.
   ![Peak of Memory metric](img/backend/mq/stackdriver_memory_peak.png)
3. Time taken of received messages.
   * Check <a href="https://cloud.google.com/logging/docs/logs-based-metrics/distribution-metrics" target="_blank">How to Create Distribution Metric</a>.
   * In the viewer panel, create a filter `jsonPayload.status=("REJECT" OR "REQUEUE" OR "ACK")` it shows only the message processed log entries
   * In the **Metric Editor** panel, set the following fields:

   | Field                                                                                                                                                                                        | Value                                                                                                                                                                                                                                                                                                                                                                           |
   |----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
   | **Name**<br/>**Add Label**<br/>**Add Label**<br/>**Units**<br/>**Type**<br/>**Field name**<br/>**Histogram buckets Type**<br/>**Start value**<br/>**Number of buckets**<br/>**Bucket width** | time_taken<br/>**Name**: topic, **Label type**: String, **Field name**: jsonPayload.extra.message_properties.”oro.message_queue.client.topic_name”<br/>**Name**: status, **Label type**: String, **Field name**: jsonPayload.status<br/>ms<br/>Distribution<br/>jsonPayload.time_taken<br/>Linear  *(setup current and below values by yours preferences)*<br/>5<br/>200<br/>10 |

   The current metrics shows total processing time for each message grouped by **Topic Name** with **Status**.
   ![Time taken metric](img/backend/mq/stackdriver_time_taken.png)

## Message Queue Dashboard

Configure “Message Queue Dashboard”, as described in <a href="https://cloud.google.com/monitoring/charts/" target="_blank">Creating and Managing Dashboard Widgets</a>, and <a href="https://cloud.google.com/logging/docs/logs-based-metrics/charts-and-alerts" target="_blank">create charts and alerts</a>.
You can monitor and be notified if something is not going as expected.

![Message Queue Dashboard](img/backend/mq/stackdriver_dashboard.png)
<!-- Frontend -->
