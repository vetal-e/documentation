#### NOTE
<a id="dev-emails"></a>

# Emails

## Email Templates

Communicating with customers is an integral part of every business. In OroPlatform, an email address is represented by the `Oro\Bundle\EmailBundle\Entity\EmailAddress` class.

When creating emails, you can reuse frequently sent text. For this, they can create templates and reuse them as needed. In addition to enabling the users to create and manage templates through the UI, every bundle can provide its own email templates.

To provide mail templates, create Twig templates for the desired format (namely HTML or plaintext) in which you can use pre-defined placeholders to define template metadata. Then you create a data fixture that implements the `AbstractEmailFixture` class. This class provides a `getEmailsDir()` method, which should return the path of the directory that contains your templates:

#### NOTE
src/Acme/Bundle/DemoBundle/Migrations/Data/ORM/EmailTemplatesFixture.php
```php
 namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

 use Oro\Bundle\EmailBundle\Migrations\Data\ORM\AbstractEmailFixture;

 class EmailTemplatesFixture extends AbstractEmailFixture
 {
     #[\Override]
     public function getEmailsDir(): string
     {
         return $this->container
             ->get('kernel')
             ->locateResource('@AcmeDemoBundle/Resources/emails');
     }
 }
```

The format of the email template is determined based on the filename extension you use:

| Extension             | Email Format   |
|-----------------------|----------------|
| `.html.twig`, `.html` | HTML           |
| `.txt.twig`, `.txt`   | Plaintext      |

HTML is used as the default format if none could be derived from the filename extension.

### Template Metadata

Additionally to the email body, the template must contain some particular parameters to add metadata to the template:

| Parameter     | Mandatory   | Description                                                                                                                  |
|---------------|-------------|------------------------------------------------------------------------------------------------------------------------------|
| `@entityName` | yes         | The fully-qualified class name of the email owner entity (for example<br/>`Acme\Bundle\DemoBundle\Entity\Applicant`).        |
| `@subject`    | yes         | The email subject.                                                                                                           |
| `@name`       | no          | The template name that is visible in the UI (the filename without the<br/>extension is used when this parameter is not set). |
| `@isSystem`   | no          | Set it to `1` to indicate that this is a system template.                                                                    |
| `@isEditable` | no          | If set to `1` and `@isSystem` is `1` too, the template can be<br/>edited in the user interface.                              |

#### NOTE
The details on template variables are described in details in the relevant email documentation.

## Sending Emails

### Create an Email

An email that is created and sent by OroPlatform is centered around the `Oro\Bundle\EmailBundle\Form\Model\Email` model class which reflects all the properties of an email.

There are two ways to create a new email:

1. [Manually create an email](#book-sending-emails-manually)
2. [Use the email builder class](#book-sending-emails-builder)

<a id="book-sending-emails-manually"></a>

#### Create an Email Manually

You can manually create an email by creating a new instance of the `Email` model class and call the setter methods for all the properties you want to be set:

```php
 namespace Acme\Bundle\DemoBundle\Controller;

 use Oro\Bundle\EmailBundle\Form\Model\Email;
 use Oro\Bundle\EmailBundle\Form\Model\EmailAttachment;
 use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

 class EmailController extends AbstractController
 {
     public function sendMailAction()
     {
         $email = new Email();

         // the sender
         $email->setFrom('chris@example.com');

         // recipient(s)
         $email->setTo(['alice@example.com', 'bob@example.com']);

         // CC recipient(s)
         $email->setCc(['dave@example.com', 'eric@example.com']);

         // BCC recipient(s)
         $email->setBcc(['ryan@example.com', 'jonathan@example.com']);

         // the subject
         $email->setSubject(...);

         // a template to create the email body, the passed template
         // must be an instance of Oro\Bundle\EmailBundle\Entity\EmailTemplate
         $email->setTemplate(...);

         // a context that will be passed to the template set with setTemplate()
         $email->setContexts([...]);

         // the email type (either html or text)
         $email->setType(...);

         // as an alternative to using a template (see above) you can also
         // directly set the email body
         $email->setBody(...);

         // email attachments (instances of  Oro\Bundle\EmailBundle\Form\Model\EmailAttachment
         $email->addAttachment($attachment);

         // ...
     }
 }
```

<a id="book-sending-emails-builder"></a>

#### The `EmailModelBuilder` Class

An alternative approach to manually creating `Email` models is to use the <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EmailBundle/Builder/EmailModelBuilder.php" target="_blank">EmailModelBuilder</a> helper class which offers several methods to create new emails based on existing data:

* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EmailBundle/Builder/EmailModelBuilder.php#L102" target="_blank">createEmailModel</a> - Create a new email or add missing data to an existing email.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EmailBundle/Builder/EmailModelBuilder.php#L136" target="_blank">createReplyEmailModel</a> - Create an email that is a response to an existing email.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EmailBundle/Builder/EmailModelBuilder.php#L169" target="_blank">createReplyAllEmailModel</a> - Create an email that responds to all recipients and the sender of an existing email.
* <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EmailBundle/Builder/EmailModelBuilder.php#L215" target="_blank">createForwardEmailModel</a> - Create a new email that forwards an existing email.

After emails have been processed (see below), they will be persisted to the database. You can create an email model based on such a persisted entity by using the useful <a href="https://github.com/oroinc/platform/blob/master/src/Oro/Bundle/EmailBundle/Builder/EmailModelBuilder.php" target="_blank">EmailModelBuilder</a> helper class:

```php
 namespace Acme\Bundle\DemoBundle\Controller;

 use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;

 class EmailController extends AbstractController
 {
     public function sendMailAction()
     {
         // ...
     }
 }
```

### Send the Email

When you create your email model, you can use the integrated mailer processor, responsible for sending the email and persisting it to the database (which also creates the needed contexts to customers, users, and so on):

```php
 namespace Acme\Bundle\DemoBundle\Controller;

 // ...

 class EmailController extends AbstractController
 {
     public function sendMailAction()
     {
         $email = new Email();
         // ...

         $processor = $this->get('oro_email.sender.email_model_sender');
         $processor->send($email);
     }
 }
```

When calling the `Oro\Bundle\EmailBundle\Sender\EmailModelSender::send` the mailer sender performs the following steps:

1. It creates a new `\Symfony\Component\Mime\Email` instance and populates it with the data from your `\Oro\Bundle\EmailBundle\Form\Model\Email` object.
2. If you did not pass an `\Oro\Bundle\EmailBundle\Entity\EmailOrigin` object, which should be used to associate the mail in the user interface, the sender would create one based on the sender address and the selected organization.
3. The email will be sent based on your application’s <a href="https://symfony.com/doc/current/reference/configuration/framework.html#mailer" target="_blank">Mailer configuration</a> (if the current user configured a custom SMTP server in their settings, the configured server would be used instead).
4. The sent email is persisted to the database storing all necessary information to be able to view it again in the future through the user interface.
5. All the persisted data is returned as an instance of the Oro\\EmailBundle\\Entity\\EmailUser.

### Receive Email Notifications

Sometimes you want to receive emails when entities of a particular class are written to the database. To achieve this, OroPlatform comes with the NotificationBundle. This bundle registers an event listener executed whenever a Doctrine entity is created, updated, or removed.

To be notified of such an event, you have to create an `Oro\Bundle\NotificationBundle\Entity\EmailNotification` that contains all the necessary information. The easiest way to register a new EmailNotification is to create data fixtures:

```php
 namespace Acme\Bundle\DemoBundle\Migrations\Data\ORM;

 use Doctrine\Common\DataFixtures\AbstractFixture;
 use Doctrine\Persistence\ObjectManager;
 use Oro\Bundle\NotificationBundle\Entity\EmailNotification;
 use Oro\Bundle\NotificationBundle\Entity\RecipientList;

 class CreateCommentNotification extends AbstractFixture
 {
     #[\Override]
     public function load(ObjectManager $manager)
     {
         $notification = new EmailNotification();

         // the FQCN of the entity
         $notification->setEntityName('Acme\Bundle\DemoBundle\Entity\Comment');

         // the event to be notified of, pre-defined event names are
         // oro.notification.event.entity_post_update, oro.notification.event.entity_post_remove
         // and oro.notification.event.entity_post_persist
         $notification->setEventName('oro.notification.event.entity_post_persist');

         // recipients must be an instance of Oro\Bundle\NotificationBundle\Entity\RecipientList
         // which represents a collection of recipients, each recipient can either be an email
         // address, a User object, or a Group object
         $recipients = new RecipientList();
         $groupRepository = $manager->getRepository('Oro\Bundle\UserBundle\Entity\Group');
         $group = $groupRepository->findOneByName('Moderator');
         $recipients->addGroup($group);

         $notification->setRecipientList($recipients);

         // the EmailTemplate that is used to render the email body
         $emailTemplateRepository = $manager->getRepository('Oro\Bundle\EmailBundle\Entity\EmailTemplate');
         $template = $emailTemplateRepository->findByName('comment_created_notification');
         $notification->setTemplate($template);

         $manager->persist($notification);
         $manager->flush();
     }
 }
```

## Email Address Owners

Each email address is owned by exactly one entity. In OroPlatform, `User` entities and `Contact` entities can be owners of an email address. Suppose you want to use the CRM to track all people applying to your company. You will then probably create an `Applicant` entity and want to associate an email address to each of them. To let your own entities own an email address, you have to follow a few steps:

1. [Create an entity responsible for storing the email address.](#book-emails-emailinterface)
2. [Create a new owner of an email address.](#book-emails-owner)
3. [Publish a provider](#book-emails-owner-provider) that makes it possible to search for the owner of a particular email address.
4. [Update the database schema and clear the cache](#book-emails-schema-cache).

<a id="book-emails-emailinterface"></a>

### Implement the Email Entity

Each entity owning an email address must have its own email entity that implements the `Oro\Bundle\EmailBundle\Entity\EmailInterface`. This interface defines four methods:

* `getEmailField()` - Returns the name of the database table column that holds the actual email address.
* `getId()` - A unique identifier to find a particular email address entity in the database.
* `getEmail()` - This method returns the actual email address.
* `getEmailOwner()` - The entity that owns a certain email address.

Sample `Email` entity:

```php
 namespace Acme\Bundle\DemoBundle\Entity;

 use Doctrine\ORM\Mapping as ORM;
 use Oro\Bundle\EmailBundle\Entity\EmailInterface;

 #[ORM\Entity]
 class ApplicantEmail implements EmailInterface
 {
     #[ORM\Id]
     #[ORM\Column(name: 'id', type: 'integer')]
     #[ORM\GeneratedValue(strategy: 'AUTO')]
     private $id;

     #[ORM\Column(type: 'string', length: 255)]
     private $email;

     #[ORM\ManyToOne(targetEntity: 'Applicant', inversedBy: 'emails')]
     private $applicant;

     #[\Override]
     public function getEmailField()
     {
         return 'email';
     }

     #[\Override]
     public function getId()
     {
         return $this->id;
     }

     #[\Override]
     public function getEmail()
     {
         return $this->email;
     }

     #[\Override]
     public function getEmailOwner()
     {
         return $this->applicant;
     }
 }
```

<a id="book-emails-owner"></a>

### An Email Owner

The entity that is the owner of the email address has to implement the `Oro\Bundle\EmailBundle\Entity\EmailOwnerInterface`:

* `getClass()` - The fully qualified class name of the entity.
* `getEmailFields()` - A list of properties of the entity that represent valid email addresses. You can specify more
  : than one property here.
* `getId()` - A unique identifier to identify a particular owner entity.
* `getFirstName()` - The first name of the email address owner. It will be used to build proper recipient names
  : when sending emails.
* `getLastName()` - The last name of the email address owner. It will be used to build proper recipient names
  : when sending emails.

For the `Applicant` entity, the implementation should be similar to the following:

```php
 namespace Acme\Bundle\DemoBundle\Entity;

 use Doctrine\Common\Collections\Collection;
 use Doctrine\ORM\Mapping as ORM;
 use Oro\Bundle\EmailBundle\Entity\EmailOwnerInterface;

 #[ORM\Entity]
 class Applicant implements EmailOwnerInterface
 {
     #[ORM\Id]
     #[ORM\Column(name: 'id', type: 'integer')]
     #[ORM\GeneratedValue(strategy: 'AUTO')]
     private $id;

     #[ORM\OneToMany(
         mappedBy: 'applicant',
         targetEntity: 'ApplicantEmail',
         cascade: ['persist'],
         orphanRemoval: true
     )]
     private $emails;

     #[ORM\Column(type: 'string', length: 255)]
     private $firstName;

     #[ORM\Column(type: 'string', length: 255)]
     private $lastName;
     private $lastName;

     #[\Override]
     public function getClass()
     {
         return 'Acme\Bundle\DemoBundle\Entity\Applicant';
     }

     #[\Override]
     public function getEmailFields()
     {
         return ['email'];
     }

     #[\Override]
     public function getId()
     {
         return $this->id;
     }

     /**
      * @return Collection
      */
     public function getEmails()
     {
         return $this->emails;
     }

     #[\Override]
     public function getFirstName()
     {
         return $this->firstName;
     }

     #[\Override]
     public function getLastName()
     {
         return $this->lastName;
     }
 }
```

<a id="book-emails-owner-provider"></a>

### Implement `EmailOwnerProviderInterface`

To make the application able to find the owner of a certain email address, you have to create a provider that implements the
`Oro\Bundle\EmailBundle\Entity\Provider\EmailOwnerProviderInterface`. This interface contains the following methods:

* `getEmailOwnerClass()` - Gets the class of an email address owner entity (the class implementing the `EmailOwnerInterface` which is the `Applicant` class in the example above).
* `findEmailOwner()` - Finds an entity object that is an owner of an email address or `null` if no such owner exists. The returned object must be an instance of the class specified by the `getEmailOwnerClass()` method.

`getOrganizations()` - Gets the list of organization IDs where an email address is used.

`getEmails()` - Gets the list of email addresses for an organization.

The provider class should then look like this:

```php
 namespace Acme\Bundle\DemoBundle\Entity\Provider;

 use Acme\Bundle\DemoBundle\Entity\Applicant;
 use Acme\Bundle\DemoBundle\Entity\ApplicantEmail;
 use Doctrine\ORM\EntityManagerInterface;
 use Oro\Bundle\BatchBundle\ORM\Query\BufferedQueryResultIterator;
 use Oro\Bundle\EmailBundle\Entity\EmailOwnerInterface;
 use Oro\Bundle\EmailBundle\Entity\Provider\EmailOwnerProviderInterface;

 class EmailOwnerProvider implements EmailOwnerProviderInterface
 {
     #[\Override]
     public function getEmailOwnerClass(): string
     {
         return Applicant::class;
     }

     #[\Override]
     public function findEmailOwner(EntityManagerInterface $em, string $email): ?EmailOwnerInterface
     {
         /** @var ApplicantEmail|null $emailEntity */
         $emailEntity = $em->getRepository(ApplicantEmail::class)->findOneBy(['email' => $email]);
         if (null === $emailEntity) {
             return null;
         }

         return $emailEntity->getEmailOwner();
     }

     #[\Override]
     public function getOrganizations(EntityManagerInterface $em, string $email): array
     {
         $rows = $em->createQueryBuilder()
             ->from(ApplicantEmail::class, 'e')
             ->select('IDENTITY(a.organization) AS id')
             ->join('e.owner', 'a')
             ->where('e.email = :email')
             ->setParameter('email', $email)
             ->getQuery()
             ->getArrayResult();

         $result = [];
         foreach ($rows as $row) {
             $result[] = (int)$row['id'];
         }

         return $result;
     }

     #[\Override]
     public function getEmails(EntityManagerInterface $em, int $organizationId): iterable
     {
         $qb = $em->createQueryBuilder()
             ->from(ApplicantEmail::class, 'e')
             ->select('e.email')
             ->join('e.owner', 'a')
             ->where('a.organization = :organizationId')
             ->setParameter('organizationId', $organizationId)
             ->orderBy('e.id');
         $iterator = new BufferedQueryResultIterator($qb);
         foreach ($iterator as $row) {
             yield $row['email'];
         }
     }
 }
```

You then need to create a service for the new `EmailOwnerProvider` class and tag it with the `oro_email.owner.provider` tag to make the application aware of the new email provider:

```yaml
 services:
     acme_demo.provider.email_owner_provider:
         class: Acme\Bundle\DemoBundle\Entity\Provider\EmailOwnerProvider
         tags:
             - { name: oro_email.owner.provider, order: 3 }
```

<a id="book-emails-schema-cache"></a>

### Refresh the Database Schema

Update the database schema and clear the application cache:

```bash
# update the database schema
php bin/console doctrine:schema:update --force

# warm up the application cache
php bin/console cache:warmup
```

#### NOTE
Business Tip

Do you want to implement smart manufacturing in your company? Learn how <a href="https://oroinc.com/b2b-ecommerce/blog/digital-transformation-in-manufacturing/" target="_blank">manufacturing digital transformation</a> can help your company get ahead.
<!-- Frontend -->
