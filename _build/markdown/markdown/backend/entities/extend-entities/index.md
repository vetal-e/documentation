<a id="book-entities-extended-entities"></a>

# Extend Entities

Common Doctrine entities have a fixed structure. This means that you cannot add additional
attributes to existing entities. Of course, one can extend an entity class and add additional
fields and associations in the subclass. However, this approach does not work anymore when an entity should be
extended by different modules.

To solve this, you can use <a href="https://github.com/oroinc/platform/tree/master/src/Oro/Bundle/EntityExtendBundle" target="_blank">EntityExtendBundle</a> which offers the following features:

* Dynamically add fields to entities through configuration.
* Users with appropriate permissions can add or remove dynamic fields from entities in the user
  interface without the assistance of a developer.
* Show dynamic fields in views, forms, and grids.
* Support for dynamic relationships between entities.

#### NOTE
It is not recommended to rely on the existence of dynamic fields in your business logic since
they can be removed by administrative users.

<a id="book-entities-extended-entities-create"></a>

## Make Entity Extended

1. Let the *entity class* implement the *ExtendEntityInterface* using the *ExtendEntityTrait*:

   #### NOTE
   src/Acme/Bundle/DemoBundle/Entity/Document.php
   ```php
   namespace Acme\Bundle\DemoBundle\Entity;

   use Doctrine\ORM\Mapping as ORM;
   use Oro\Bundle\EntityConfigBundle\Metadata\Attribute\Config;
   use Oro\Bundle\EntityExtendBundle\Entity\ExtendEntityInterface;
   use Oro\Bundle\EntityExtendBundle\Entity\ExtendEntityTrait;

   /**
    * ORM Entity Document.
    */
   #[ORM\Entity]
   #[ORM\Table(name: 'acme_demo_document')]
   #[Config]
   class Document implements ExtendEntityInterface
   {
     use ExtendEntityTrait;
   }
   ```

   2. Add new fields using a migration script:

   ```php
   <?php

   namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_1;

   use Doctrine\DBAL\Schema\Schema;
   use Oro\Bundle\EntityBundle\EntityConfig\DatagridScope;
   use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
   use Oro\Bundle\MigrationBundle\Migration\Migration;
   use Oro\Bundle\MigrationBundle\Migration\QueryBag;

   class AddDocumentRatingColumn implements Migration
   {
       #[\Override]
       public function up(Schema $schema, QueryBag $queries)
       {
           $table = $schema->getTable('acme_demo_document');
           $table->addColumn(
               'document_rating',
               'integer',
               ['oro_options' => [
                   'extend' => [
                       'is_extend' => true,
                       'owner' => ExtendScope::OWNER_CUSTOM
                   ],
                   'entity' => ['label' => 'Document rating'],
                   'datagrid' => ['is_visible' => DatagridScope::IS_VISIBLE_TRUE]
               ]]
           );
       }
   }
   ```

   The example above adds a new column `document_rating`. The third parameter configures the column
   as an extended field. The `ExtendScope::OWNER_CUSTOM` owner in the `oro_options` key
   indicates that the column was added dynamically. It will be visible and configurable in the UI.

   Note that this field is neither present in the `Document` entity class nor in the
   `ExtendDocument` class in your bundle, but it will only be part of the `ExtendDocument` class that
   will be generated in your application cache.

   1. Finally, load the changed configuration using the `oro:migration:load` command:
      ```bash
      php bin/console oro:migration:load
      ```

   #### NOTE
   You can add, modify, and remove custom fields in the UI under *System > Entities > Entity Management*.

   <!-- Apply Changes
   -------------

   The following command updates the database schema and all related caches to reflect changes made in extended entities:

   .. code-block:: bash

       php bin/console oro:entity-extend:update

   The ``dry-run`` can be used to show changes without applying them, for example:

   .. code-block:: bash

       php bin/console oro:entity-extend:update --dry-run -->

   <a id="book-entities-extended-entities-add-fields"></a>

   ## Add Entity Fields

   You may require to customize the default Oro entities to meet the needs of your application.

   Let us customize the User entity to store the date when a contact becomes a member of your company’s partner network.
   As an illustration, we will use the User entity from a custom DemoBundle.

   To achieve this, add a new field `partnerSince` to store the date and time of when a contact joined your network.
   To add the field, create a migration:

   ```php
   <?php

   namespace Acme\Bundle\DemoBundle\Migrations\Schema\v1_2;

   use Doctrine\DBAL\Schema\Schema;
   use Oro\Bundle\EntityExtendBundle\EntityConfig\ExtendScope;
   use Oro\Bundle\MigrationBundle\Migration\Migration;
   use Oro\Bundle\MigrationBundle\Migration\QueryBag;

   class AddPartnerSinceToOroUser implements Migration
   {
       #[\Override]
       public function up(Schema $schema, QueryBag $queries)
       {
           $table = $schema->getTable('oro_user');
           $table->addColumn('partner_since', 'datetime', [
               'oro_options' => [
                   'extend' => [
                       'is_extend' => true,
                       'owner' => ExtendScope::OWNER_CUSTOM,
                       'nullable' => true,
                       'on_delete' => 'SET NULL'
                   ],
                   'entity' => ['label' => 'Partner since']
               ],
           ]);
       }
   }
   ```

   #### NOTE
   Please note that the entity that you add a new field to must have the `#[Config]` attribute
   and should extend an Extend class.

   The important part in this migration (which is different from common Doctrine migrations) is the `oro_options` key.
   It is passed through the `options` argument of the `addColumn()` method:

   ```php
   // ...
            $table->addColumn('partnerSince', 'datetime', [
                'oro_options' => [
                    'extend' => [
                        'is_extend' => true,
                        'owner' => ExtendScope::OWNER_CUSTOM,
                        'nullable' => true,
                        'on_delete' => 'SET NULL'
                    ],
                ],
            ]);
   // ...
   ```

   All options nested under this key are handled outside of the usual Doctrine migration workflow.

   When the EntityExtendBundle of the OroPlatform finds the `extend` key, it generates an intermediate class
   with getters and setters for the defined fields, thus making them accessible from every part of the code.
   The intermediate class is generated automatically based on the configured data when the application cache is warmed up.

   The `owner` attribute can have the following values:

   * `ExtendScope::OWNER_CUSTOM` — The field is user-defined, and the core system should handle how the field appears in grids, forms, etc. (if not configured otherwise).
   * `ExtendScope::OWNER_SYSTEM`— Nothing is rendered automatically, and the developer must explicitly specify how to show the field in different parts of the system (grids, forms, views, etc.).

   #### NOTE
   For more default attribute set settings for Extend Entities, see <a href="https://doc.oroinc.com/backend/configuration/annotation/config-field" target="_blank">#[ConfigField]</a>.

   <a id="book-entities-extended-entities-add-enum-fields"></a>

   ## Add Enum Option Set Fields

   The option set fields can be used to choose one or more options from a predefined set of options.
   The [Option Set Fields](enums.md#book-entities-extended-entities-enums) section provides detailed information on
   how to add such fields.

   <a id="book-entities-extended-entities-add-relationships"></a>

   ## Add Entity Relationships

   Adding relationships between entities is a common but, in some cases, complex task.
   The [Extended Associations](associations.md#book-entities-extended-entities-associations)
   and [Multi-Target Extended Associations](multi-target-associations.md#book-entities-extended-entities-multi-target-associations)
   sections provide detailed information on how to add different kinds of relationships.

   ## Console Commands

   * Clear cache.

     Use the `oro:entity-extend:cache:clear` command to clear extended entity cache.
     ```none
     php bin/console oro:entity-extend:cache:clear
     ```
   * Skip warming up cache.

     Use the `--no-warmup` option to skip warming up cache after cleaning:
     > ```none
     > php bin/console oro:entity-extend:cache:clear --no-warmup
     > ```
   * Warm up cache.

     Use the `oro:entity-extend:cache:warmup` command to warm up extended entity cache and its related caches (Doctrine metadata, Doctrine proxy classes for extended entities, cache of entity aliases).
     ```none
     php bin/console oro:entity-extend:cache:warmup
     ```

     The `--cache-dir` option can be used to override the default cache directory location.
     ```none
     php bin/console oro:entity-extend:cache:warmup --cache-dir=<path>
     ```
   * Update schema.

     Use the `oro:entity-extend:update-schema` command to update database schema for extend entities.
     ```none
     php bin/console oro:entity-extend:update-schema
     ```

     The `--dry-run` option can be used to print the changes without applying them:
     ```none
     php bin/console oro:entity-extend:update --dry-run
     ```

   #### WARNING
   Schema changes are permanent and cannot be easily rolled back. We recommend that developers back up data before any database schema change if changes have to be rolled back.

   #### NOTE
   Business Tip

   Looking for a way to leverage online commerce? Here’s everything you need to know about a <a href="https://oroinc.com/oromarketplace/b2b-marketplace/" target="_blank">B2B online marketplace</a> and what makes it work.

   * [Option Enum Set Fields](enums.md)
   * [Extended Associations](associations.md)
   * [Multi-Target Extended Associations](multi-target-associations.md)
   * [Serialized Fields](serialized-fields.md)
   * [Validation for Extended Fields](validation.md)
   * [Define Custom Form Type for Fields](define-custom-form-type.md)
   * [Extending the Extended Field Rendering](extending-rendering.md)

   <!-- Frontend -->
