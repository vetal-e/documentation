<a id="dev-guide-system-cron-jobs"></a>

# Cron

More often than not, it is essential to run regular time-based background jobs in business applications. These jobs can be maintenance tasks, such as checking for updates or synchronizing data between integrated systems, and business-related tasks, such as generating reports, sending emails, or making timely-based shifts in tasks determined by your business process flows.

The nature and algorithms of these timely-based tasks can be diverse and complicated, and the obvious way of implementing these jobs is to create specific program components.

Therefore, to strengthen the task to create and schedule such components, OroPlatform provides the OroCronBundle . With its help, it is considerably easier to run Symfony Console commands through cronjobs (on UNIX-based operating systems) or the Windows task scheduler.

The OroCronBundle provides two interfaces that help to implement console commands that should be executed by the cron:

- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Command/CronCommandScheduleDefinitionInterface.php" target="_blank">CronCommandScheduleDefinitionInterface</a> allows defining the console command along with its schedule in a crontab compatible string in the command class.
- <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Command/CronCommandActivationInterface.php" target="_blank">CronCommandActivationInterface</a> allows defining a conditional logic for the cron command.

The **oro:cron:definitions:load** command scans for all commands from the oro:cron namespace that implements the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Command/CronCommandScheduleDefinitionInterface.php" target="_blank">CronCommandScheduleDefinitionInterface</a>. For each detected command, a new <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Entity/Schedule.php" target="_blank">Schedule</a> entry is created and saved into the database. This command runs on install and update, and can also be run manually if some cron commands or command definitions are changed.

The second command is **oro:cron**. It takes all schedules from the database (created by the oro:cron:definitions:load command) and adds the commands that are due to the Message Queue. This command should run every minute.

#### NOTE
Please notice that oro:cron:definitions:load removes all previously loaded commands from the database. So if other commands add cron commands to the db (such as oro:workflow:definition:load), they should be run after **oro:cron:definitions:load**.

## Setup and Configuration

To run a set of commands from your application regularly, configure your system to run the **oro:cron** command every minute.

* For UNIX-based systems, set up a crontab entry, as illustrated below:
  > ```none
  > */1 * * * * /path/to/php /path/to/bin/console oro:cron --env=prod > /dev/null
  > ```

  > #### NOTE
  > Some OS flavors require a username (usually root) in the crontab entry:
  > ```none
  > */1 * * * * root /path/to/php /path/to/bin/console oro:cron --env=prod > /dev/null
  > ```
* For Windows, use the Control Panel to configure the Task Scheduler to do the same.
  > #### NOTE
  > This entry in the crontab does not presuppose the execution of cron commands every minute. The oro:cron command only ensures that the actual commands are added to the scheduler which in its turn makes sure that they are executed at desired time.

<a id="dev-cookbook-system-cron-create-commands"></a>

## Scheduled Commands in OroPlatform

A scheduled command in OroPlatform is a regular Symfony console command that implements additional <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Command/CronCommandScheduleDefinitionInterface.php" target="_blank">CronCommandScheduleDefinitionInterface</a> and has the **oro:cron** namespace.

Implementing <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Command/CronCommandScheduleDefinitionInterface.php" target="_blank">CronCommandScheduleDefinitionInterface</a> requires the implementation of the **getDefaultDefinition()** method. It returns the <a href="http://www.unix.com/man-page/linux/5/crontab/" target="_blank">crontab compatible</a> description of when the command should be executed. For example, if a command should run every day five minutes after midnight, the appropriate
value is **5 0 \* \* \***.

```php
namespace Acme\Bundle\DemoBundle\Command;

use Oro\Bundle\CronBundle\Command\CronCommandScheduleDefinitionInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class SomeCronCommand extends Command implements CronCommandScheduleDefinitionInterface
{
    protected static $defaultName = 'oro:cron:acme_demo_some';

    #[\Override]
    public function getDefaultDefinition(): string
    {
        return '5 0 * * *';
    }

    #[\Override]
    protected function configure()
    {
        // ...
    }

    #[\Override]
    protected function execute(InputInterface $input, OutputInterface $output)
    {
        // ...
    }
}
```

### Conditional Activation of Cron Commands

By default, all cron commands are executed every time when the cron triggers it. But sometimes it is required
to execute a command only when certain conditions are met.
In this case a cron command should implement <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Command/CronCommandActivationInterface.php" target="_blank">CronCommandActivationInterface</a> interface and provide the custom activation logic in the **isActive()** method.

### Synchronous Cron Commands

By default, **all cron commands are executed asynchronously** by sending a message to the queue.

Sometimes it is necessary to execute a cron command **immediately** when cron triggers it, without sending the message
to the queue.

To do this, a cron command should implement the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/CronBundle/Command/SynchronousCommandInterface.php" target="_blank">SynchronousCommandInterface</a> interface. In this case, the command will be executed as a background process.

#### NOTE
Please note that the synchronous commands must be designed well-performed and should not block process execution as it may affect scheduled execution of other commands.

### Scheduling Cron Commands in DB

After creating the cron commands classes, run the **oro:cron:definitions:load** command to schedule the created
command in the DB. After that, the cron command will be ready to evaluate and execute it during the next **oro:cron** command tick.

**Related Topics**

* View the List of Scheduled Tasks in UI

<!-- Frontend -->
