# #[TitleTemplate]

This attribute is used to configure the template that is used to render the page title when the
controller tagged with this attribute is accessed:

```php
// ...
use Oro\Bundle\NavigationBundle\Attribute\TitleTemplate;

#[TitleTemplate("The page title")]
public function demoAction()
{
    // ...
}
```

You can use arbitrary placeholders here which you then have to replace later on in your template
using the `oro_title_set()` Twig function:

```html+jinja
{% oro_title_set({params : {"%username%": fullname }}) %}
```
