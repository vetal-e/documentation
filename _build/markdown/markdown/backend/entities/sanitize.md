<a id="dev-sanitize"></a>

# Data Sanitization

Data sanitization is necessary in cases where sensitive data is not intended to be exposed when it is distributed to environments other than the live one, in particular, the developer’s one.

The sanitization mechanism enables developers to define sanitization rules or raw SQL queries for entities and their fields, allowing those elements to be dumped as ready-to-use SQL queries. These queries are intended for use with database copies that can be distributed to potentially insecure environments.

#### NOTE
The supported SQL syntax is for PostgreSQL only. The PostgreSQL server will validate the generated SQL queries for syntax errors. No semantic analysis is performed, so the column and table names specified in raw SQL queries are their authors’ responsibility.

## How to Get a Dump of DB Sanitizing SQL Queries

To get a list of sanitizing SQL queries, run the following command:

```bash
php bin/console oro:sanitize:dump-sql
```

In this case, SQLs are output to the console.

Dump can be made directly to a file by specifying it as a command argument:

```bash
php bin/console oro:sanitize:dump-sql /tmp/sanitize.sql
```

The generated dump will look as follows both in the file and in the console:

```none
-- !!!Exercise caution when using TRUNCATE queries with CASCADE options --
TRUNCATE "acme_blog_post";
DELETE FROM "acme_demo_question" WHERE "created_at" < (NOW() - INTERVAL '5 days');
UPDATE "acme_demo_question" SET "due_date"=CURRENT_TIMESTAMP;
DELETE FROM "acme_demo_sms" WHERE "id" NOT IN(SELECT "id" FROM "acme_demo_sms" ORDER BY "id" DESC LIMIT 3);
UPDATE "acme_demo_sms" SET serialized_data = serialized_data || jsonb_build_object('delivery_date', CURRENT_TIMESTAMP(0));
```

This particular example is aimed to truncate the **acme_blog_post** table, then reduce the number of records in **acme_demo_question**, **acme_demo_sms**, and then hide the dates information that should not be exposed. The example contains only a few queries that follow the sanitizing configuration specified in the [Sanitizing Rules Defined in Files]()  and  [Sanitizing Rules Defined in the Entity Configuration]() topics below.

The generated full list of SQL queries will also include queries built on configurations defined in the core bundles of the Oro application.

#### NOTE
If there are any syntax errors in the resulting SQL queries, the customer will be notified. The queries, however, will not be included in the file, even if such file is specified,  but will be output to the console with an indication of invalid queries.

#### NOTE
If there are any errors in the rule configuration caused by incorrect field or entity names, or if a rule is assigned to a field that it cannot process, then the console output will identify the issues and prevent any queries from being executed.

## Sanitizing Rule Sources

There are two source options to specify sanitizing rules.

The first option is to store the rule configuration in the **santize.yml** files, which are placed in **Resources/config/oro** by convention. The [bundleless](../architecture/bundle-less-structure.md#dev-backend-architecture-bundle-less-structure) approach is also supported for this case. If a rule configuration refers to a specific entity or field, the data from the last file read takes precedence over any previous files.

The second option is to store the rule configuration within the entity and its field configuration in a dedicated scope. This approach is more complicated to maintain, but it ensures that the sanitized configuration is fixed. The configuration set in this way has priority over the one read from a file.

The file-based approach is easier to maintain and is preferable in most cases. This is the only option if the database table isn’t bound to the entity.

### Sanitizing Rules Defined in Files

#### NOTE
src/Acme/Bundle/DemoBundle/Resources/config/oro/sanitize.yml
```yaml
oro_sanitize:
    raw_sqls:
        - '-- Sample comment as unbound raw SQL'
    entity:
        acme_blog_post: truncate
        # example of specifying an entity class
        Acme\Bundle\DemoBundle\Entity\Document:
            fields:
                subject: 
                    rule: md5
                    rule_options:
                        length: 10
                # example of specifying a field name but not a column name
                dueDate: date
        acme_demo_question:
            raw_sqls:
                - DELETE FROM "acme_demo_question" WHERE "created_at" < (NOW() - INTERVAL '5 days')
            fields:
                subject:
                    raw_sqls:
                        - 'UPDATE "acme_demo_question" SET "subject"=SUBSTRING(MD5("subject" || RANDOM()::TEXT) FROM 1 FOR 10)'
                due_date: date
        acme_demo_sms:
            rule: keep_last_rows
            rule_options:
                rows_count: 3
            fields:
                message: str_reverse
```

The **raw_sqls** node under the **oro_sanitize** node lists sanitizing SQL queries without binding them to any entity or field.

Items keyed by entity class or table name go under the **entity** node. Each item can have its own **raw_sqls** items, rule definition, and the **fields** section. Such an item can also have only one string value that defines the sanitizing rule. This case is suggested as having a **rule** value set. Note that at least one of the **raw_sqls**, **rule**, or **fields** values must be set.

The **fields** items are keyed with the field or column name. The inner of the field element is almost identical to that of the entity element. It has **raw_sqls** elements and a rule definition. Also, as an entity item, it can only have a string value that defines the sanitizing rule. This case is suggested as having a **rule** value set. At least one of the **raw_sqls**, **rule** values must be set.

The **rule** values are checked against the list of registered rule processors both for entities and fields.

#### NOTE
If the **rule** or **raw_sqls** configurations for an entity or a field appear in a file while it has already been read from another file, then the new configuration will overwrite the old one.

### Sanitizing Rules Defined in the Entity Configuration

This can be done through schema migration or by defining rules in the entity’s configuration annotation.

Example of adding sanitizing rules to the entity configuration via migration:

```php
<?php

namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_12;

use Acme\Bundle\DemoBundle\Entity\Sms;
use Doctrine\DBAL\Schema\Schema;
use Oro\Bundle\EntityConfigBundle\Migration\UpdateEntityConfigFieldValueQuery;
use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
use Oro\Bundle\EntitySerializedFieldsBundle\Migration\Extension\SerializedFieldsExtension;
use Oro\Bundle\EntitySerializedFieldsBundle\Migration\Extension\SerializedFieldsExtensionAwareInterface;
use Oro\Bundle\MigrationBundle\Migration\Migration;
use Oro\Bundle\MigrationBundle\Migration\QueryBag;

class AddFieldsWithSanitizingRulesMigration implements Migration, SerializedFieldsExtensionAwareInterface
{
    protected SerializedFieldsExtension $serializedFieldsExtension;

    #[\Override]
    public function setSerializedFieldsExtension(SerializedFieldsExtension $serializedFieldsExtension)
    {
        $this->serializedFieldsExtension = $serializedFieldsExtension;
    }

    #[\Override]
    public function up(Schema $schema, QueryBag $queries)
    {
        $table = $schema->getTable('acme_demo_sms');
        // adding new field as extended one with sanitzing options
        $table->addColumn(
            'moderation_notes',
            'text',
            [
                'oro_options' => [
                    'extend' => ['is_extend' => true, 'owner' => ExtendScope::OWNER_CUSTOM],
                    // sanitizing rule configuration
                    'sanitize' => [
                        'raw_sqls' => ['UPDATE "acme_demo_sms" SET "moderation_notes"=NULL']
                    ]
                ]
            ]
        );

        // adding new serialized data field with sanitzing options
        $this->serializedFieldsExtension->addSerializedField(
            $table,
            'delivery_date',
            'datetime',
            [
                'extend' => ['owner' => ExtendScope::OWNER_CUSTOM],
                'entity' => ['label' => 'Delivery Date'],
                // sanitizing rule configuration
                'sanitize' => ['rule' => 'date']
            ]
        );

        // adding sanitizing options to existing field
        $queries->addQuery(
            new UpdateEntityConfigFieldValueQuery(Sms::class, 'fromContact', 'sanitize', 'rule', 'str_reverse')
        );
    }
}
```

This example covers cases where new ordinary and serialized fields are created with sanitizing configuration, and where an existing field is assigned with sanitizing configuration. In cases where the rule requires additional setup, the **rule_options** can also be added. Please note that when updating an existing field, a separate **UpdateEntityConfigFieldValueQuery** instance is required for each configuration value that needs to be updated.

Example of adding sanitizing rules to a newly created entity using a config annotation:

```php
namespace Acme\Bundle\DemoBundle\Entity;
use Doctrine\ORM\Mapping as ORM;
    // ...
use Oro\Bundle\EntityExtendBundle\Entity\ExtendEntityInterface;
use Oro\Bundle\EntityExtendBundle\Entity\ExtendEntityTrait;
    // ...
/**
 * ORM Entity Sms.
 */
#[ORM\Entity(repositoryClass: 'Acme\Bundle\DemoBundle\Entity\Repository\SmsRepository')]
class Sms implements
    ExtendEntityInterface
{
    // ...
    use ExtendEntityTrait;
    // ...
    #[ORM\Column(name: 'message', type: 'text', nullable: true)]
    #[ConfigField(defaultValues: ['dataaudit' => ['auditable' => true], 'sanitize' => ['rule' => 'str_reverse']])]
    private $message;
    // ...
}
```

#### NOTE
Rules are not applied to relations, but only to scalar fields or serialized ones.

## Predefined Sanitizing Rules

Predefined rule processors for entities:

* **truncate** - builds a table truncation query. The rule has no options.
* **truncate_cascade** - builds a table truncation query with a cascade option. The rule has no options.

Predefined rule processors for fields:

* **date** - builds a query to replace the field value with the current date. The rule can only be applied to the date field. The rule has no options.
* **md5** - builds a query to replace a field value with its own MD5 hash, salted with a random value. The rule can only be applied to the string (text, varchar) field. The rule has the **length** option. If none is specified, then the read length of the field is used.
* **email** - builds a query to replace the email’s server name with either an MD5 hashed server name or a custom server name if specified in the application’s configuration. Additionally, if the primary key value of the DB record is numeric, the query salts the mailbox name with the key. The rule can only be applied to the string (text, varchar) field. The rule has no options.
* **set_null** - builds a query to replace a field value with a null. There are no field-type restrictions. The rule has no option.
* **digits_mask** - builds a query to replace the field value with a **phone** number mask. The mask should look like the following: **1 (800) XXX-XXXX**. The **X** symbol in the mask will be replaced with one of the digits from the random value based on the 10000000 number. The length of the value will correspond to the number of **X** symbols in the mask. The rule can only be applied to the string (text, varchar) field. The rule has a **mask** option, as shown in the example above.
* **generic_phone** - is a special case of a **digits_mask** rule with a predefined mask specified in the application configuration. The rule has no options.

The Oro application settings example for the **email** and **generic_phone** rules:

```php
oro_sanitize:
    # Custom email domain for the 'email' sanitizing rule
    custom_email_domain: example.com
    # Digits mask for the 'generic_phone' sanitizing rule. The default value is shown
    generic_phone_mask: (XXX) XXX-XXXX
```

## Guessing Field Sanitizing Rules

If no sanitizing rule is directly specified for a field, the rule processor’s guessing mechanism tries to find one.

The sanitize functionality comes with the following pre-defined field rule processor guessers:

* Email field guesser. It relies on the field’s type, which must be a string and its name. The name should be either the word **email** itself or part highlighted with camel case or under case. For example, **email**, **emailSecond**, **email_Third**, **new_email**, or **anotherEmail**. The guessed rule processor is **email**.
* Full name parts guesser. These parts are the middle name and last name. It relies on the **middleName** and **lastName** field names and specific interfaces implemented by the processed entity. The guessed rule processor is **md5**.
* Crypted string field guesser. It relies on the **crypted_string** field type, which is commonly used to extend integration data tables. The guessed rule process is **md5**.

## Custom Sanitizing Rule Processor

If it’s necessary to define repeating actions in relation to sanitizing actions, the custom sanitizing rule processors can be implemented instead of writing raw SQL queries.

### Custom Entity Sanitizing Rule Processor

An example of the entity rule sanitizing processor is keeping the last added rows.

To define a custom rule processor, add a service that implements **Oro\\Bundle\\SanitizeBundle\\RuleProcessor\\Entity\\ProcessorInterface** and has the **oro_sanitize.entity_rule.processor** tag:

```yaml
services:
    acme_demo.sanitize.entity_rule.keep_last_processor:
        class: Acme\Bundle\DemoBundle\Sanitize\RuleProcessor\Entity\KeepLastRowsProcessor
        arguments:
            - '@doctrine.dbal.default_connection'
        tags:
            - { name: 'oro_sanitize.entity_rule.processor'}
```

The sanitizing rule implementation:

```php
<?php

namespace Acme\Bundle\DemoBundle\Sanitize\RuleProcessor\Entity;

use Doctrine\DBAL\Connection;
use Doctrine\ORM\Mapping\ClassMetadata;
use Oro\Bundle\SanitizeBundle\RuleProcessor\Entity\ProcessorInterface;

/**
 * Sanitizing rule processor for an entity that keeps last added rows regarding primary key.
 */
class KeepLastRowsProcessor implements ProcessorInterface
{
    private const KEEP_LAST_ROWS_DEFAULT_COUNT = 10;

    public function __construct(private Connection $connection)
    {
    }

    #[\Override]
    public static function getProcessorName(): string
    {
        return 'keep_last_rows';
    }

    #[\Override]
    public function getSqls(ClassMetadata $metadata, array $sanitizeRuleOptions = []): array
    {
        try {
            $idFieldType = $metadata->getFieldMapping($metadata->getSingleIdentifierFieldName())['type'];
            if (!in_array($idFieldType, ['integer', 'bigint', 'smallint'], true)) {
                throw new \RuntimeException();
            }
        } catch (\Throwable $e) {
            throw new \Exception(spritnf(
                "Could not detect single identifier numeric field name for '%s' entity",
                $metadata->getName()
            ));
        }

        return [sprintf(
            'DELETE FROM %1$s WHERE %2$s NOT IN(SELECT %2$s FROM %1$s ORDER BY %2$s DESC LIMIT %3$s)',
            $this->connection->quoteIdentifier($metadata->getTableName()),
            $this->connection->quoteIdentifier($metadata->getSingleIdentifierFieldName()),
            ((int) ($sanitizeRuleOptions['rows_count'] ?? 0)) ?: self::KEEP_LAST_ROWS_DEFAULT_COUNT
        )];
    }
}
```

An entity sanitizing rule processor must provide an implementation of the following routines:

* It must supply the name of the processor. This is the responsibility of the **getProcessorName** static method.
* It must return valid SQL queries for an entity. This is the responsibility of the **getSqls** methods.

### Custom Field Sanitizing Rule Processor

An example of a field rule sanitizing processor is to implement a simple string reverse action.

To define a custom rule processor, add a service that implements **Oro\\Bundle\\SanitizeBundle\\RuleProcessor\\Field\\ProcessorInterface** and has the **oro_sanitize.field_rule.processor** tag:

```yaml
services:
    acme_demo.sanitize.field_rule.reverse_processor:
        class: Acme\Bundle\DemoBundle\Sanitize\RuleProcessor\Field\ReverseProcessor
        parent: oro_sanitize.field_rule.generic_processor
        tags:
            - { name: 'oro_sanitize.field_rule.processor' }
```

To simplify the definition of a sanitizing rule processor service, the parent abstract definition **oro_sanitize.field_rule.generic_processor** has been prepared. It proposes a common provision for dependency injection.

The sanitizing rule implementation:

```php
<?php

namespace Acme\Bundle\DemoBundle\Sanitize\RuleProcessor\Field;

use Doctrine\ORM\Mapping\ClassMetadata;
use Oro\Bundle\SanitizeBundle\RuleProcessor\Field\Helper\ProcessorHelper;
use Oro\Bundle\SanitizeBundle\RuleProcessor\Field\JsonBuildPairsPostProcessor;
use Oro\Bundle\SanitizeBundle\RuleProcessor\Field\ProcessorInterface;
use Oro\Bundle\SanitizeBundle\RuleProcessor\Field\SerializeFieldCheckerTrait;

/**
 * Reverse string sanitizing rule processor for a field.
 */
class ReverseProcessor implements ProcessorInterface
{
    use SerializeFieldCheckerTrait;

    public function __construct(
        private JsonBuildPairsPostProcessor $jsonBuildPairsPostProcessor,
        private ProcessorHelper $helper
    ) {
    }

    #[\Override]
    public static function getProcessorName(): string
    {
        return 'str_reverse';
    }

    #[\Override]
    public function getIncompatibilityMessages(
        string $fieldName,
        ClassMetadata $metadata,
        array $sanitizeRuleOptions = []
    ): array {
        if (!$this->helper->isStringField($fieldName, $metadata)) {
            return [sprintf(
                ProcessorHelper::NON_DATE_FIELD_PROCESSED,
                $fieldName,
                $metadata->getName(),
                self::getProcessorName()
            )];
        }

        return [];
    }

    #[\Override]
    public function getSqls(
        string $fieldName,
        ClassMetadata $metadata,
        array $sanitizeRuleOptions = []
    ): array {
        $quotedColumnName = $this->helper->getQuotedColumnName($fieldName, $metadata);
        return [sprintf(
            "UPDATE %s SET %s=%s",
            $this->helper->quoteIdentifier($metadata->getTableName()),
            $quotedColumnName,
            $this->getUpdateSqlValue($quotedColumnName)
        )];
    }

    protected function doPrepareSerializedFieldUpdate(
        string $fieldName,
        ClassMetadata $metadata,
        array $sanitizeRuleOptions = []
    ): void {
        $updateSqlValue = $this->getUpdateSqlValue(
            sprintf("serialized_data->>%s", $this->helper->quoteString($fieldName))
        );
        $this->jsonBuildPairsPostProcessor
            ->addJsonBuildPairForTable($metadata->getTableName(), $fieldName, $updateSqlValue);
    }

    private function getUpdateSqlValue(string $quotedColumnName): string
    {
        return sprintf('REVERSE(%s)', $quotedColumnName);
    }
}
```

A field sanitizing rule processor must provide an implementation of the following routines:

* It must supply the name of the processor. This is the responsibility of the **getProcessorName** static method.
* It must return information about incompatibilities. This is the responsibility of the **getIncompatibilityMessages** method.
* It must prepare a valid SQL update part for the serialized field. This is the responsibility of the **prepareSerialisedFieldUpdate** method.
* It must return valid SQL queries for scalar fields. This is the responsibility of the **getSqls** methods.

From the above example, the **prepareSerialisedFieldUpdate** method is wrapped by the **SerializeFieldCheckerTrait** trait method. The trait method performs additional validation to check whether the field being processed is serialized or not. However, this extra validation is unnecessary and is just an additional protection against any misuse of a field rule processor.

It is also possible to reconfigure existing field rule processors using a dedicated wrapping component. Such processors can be defined in the following way:

```yaml
services:
    acme_demo.sanitize.field_rule.toll_free_phone_processor:
        class: Oro\Bundle\SanitizeBundle\RuleProcessor\Field\WrappedProcessor
        arguments:
            - '@Oro\Bundle\SanitizeBundle\RuleProcessor\Field\DigitsMaskProcessor'
        calls:
            - ['setOptions', [{mask: '1 (800) XXX-XXX-XXXX'}]]
        tags:
            - { name: 'oro_sanitize.field_rule.processor', processor_name: 'toll_free_phone' }
```

This example defines the toll-free phone-like random number generator.

It’s important to name the wrapping processor in the **processor_name** tag’s property instead of calling the unsuggested **getProcessorName** method.
