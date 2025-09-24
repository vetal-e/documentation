<a id="acl-ancestor"></a>

# #[AclAncestor]

This attribute is used to protect a controller based on an existing access control list. The ID of
the parent access control list is passed as the only argument:

```php
// ...
use Oro\Bundle\SecurityBundle\Attribute\AclAncestor;

#[AclAncestor("an_acl_id")]
public function demoAction()
{
    // ...
}
```
