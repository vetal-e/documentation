<a id="backend-security-bundle-csrf"></a>

# CSRF Protection

<a href="https://owasp.org/www-community/attacks/csrf" target="_blank">Cross-Site Request Forgery (CSRF)</a> is an attack that forces an end user to execute unwanted actions on a web application in which they are currently authenticated.

## AJAX Request CSRF Protection

To protect controllers against CSRF, use AJAX #[CsrfProtection] attribute. You can use it for the whole controller or individual actions.

<a href="https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html#double-submit-cookie" target="_blank">Double Submit Cookie</a> technique used for AJAX request protection, each AJAX request must have an X-CSRF-Header header with a valid token value, this header is added by default to all AJAX requests. The current token value is stored in the cookie \_csrf for HTTP connections and https-_csrf for HTTPS.

**Controller level protection**

#### NOTE
src/Acme/Bundle/DemoBundle/Controller/FavoriteController.php
```php
<?php

namespace Acme\Bundle\DemoBundle\Controller;

use Acme\Bundle\DemoBundle\Entity\Favorite;
use Oro\Bundle\EntityBundle\ORM\DoctrineHelper;
use Oro\Bundle\SecurityBundle\Attribute\Acl;
use Oro\Bundle\SecurityBundle\Attribute\AclAncestor;
use Oro\Bundle\SecurityBundle\Attribute\CsrfProtection;
use Oro\Bundle\SecurityBundle\ORM\Walker\AclHelper;
use Symfony\Bridge\Twig\Attribute\Template;
use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\Routing\Attribute\Route;
use Symfony\Component\Security\Acl\Voter\FieldVote;
use Symfony\Component\Security\Core\Exception\AccessDeniedException;

/**
 * Contains CRUD actions for Favorite
 */
    #[CsrfProtection]
#[Route(path: '/favorite', name: 'acme_demo_favorite_')]
class FavoriteController extends AbstractController
{
}
```

**Action level protection**

```php
<?php

namespace Acme\Bundle\DemoBundle\Controller;

    #[Route(path: '/custom', name: 'custom')]
    #[CsrfProtection]
    #[Template('@AcmeDemo/Favorite/index.html.twig')]
    #[Acl(
        id: 'acme_demo_favorite_custom',
        type: 'entity',
        class: 'Acme\Bundle\DemoBundle\Entity\Favorite',
        permission: 'VIEW'
    )]
    public function customAction(): array
    {
        return ['entity_class' => Favorite::class];
    }
```

<!-- Frontend -->
