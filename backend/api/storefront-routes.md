<a id="web-api-storefront-routes"></a>

# Storefront Routes

Storefront API has an API resource `routes` that returns the information about the storefront URLs.
This information includes a resource type and the relative URL of an API resource that you can use to get the data.

Two types of resolvers are used to provide this information:

> - the resource type resolver, that is represented by <a href="https://github.com/oroinc/customer-portal/blob/master/src/Oro/Bundle/FrontendBundle/Api/ResourceTypeResolverInterface.php" target="_blank">ResourceTypeResolverInterface</a>;
> - the API URL resolver that is represented by <a href="https://github.com/oroinc/customer-portal/blob/master/src/Oro/Bundle/FrontendBundle/Api/ResourceApiUrlResolverInterface.php" target="_blank">ResourceApiUrlResolverInterface</a>.

The resource type resolvers should be registered in the service container with a tag `oro_frontend.api.resource_type_resolver`,
and optionally, the `routeName` tag attribute can be used to specify the route for which the resolver is applicable.

The API URL resolvers should be registered in the service container with a tag `oro_frontend.api.resource_api_url_resolver`,
and optionally, the `routeName` tag attribute can be used to specify the route for which the resolver is applicable.

An example of resolvers registration in the services_api.yml file:

```yaml
services:
    oro_cms.api.resource_type_resolver.landing_page:
        class: Oro\Bundle\FrontendBundle\Api\ResourceTypeResolver
        arguments:
            - 'landing_page' # the resource type should be returned for the route oro_cms_frontend_page_view
        tags:
            - { name: oro_frontend.api.resource_type_resolver, routeName: oro_cms_frontend_page_view }

    oro_cms.api.resource_api_url_resolver.landing_page:
        class: Oro\Bundle\FrontendBundle\Api\ResourceRestApiGetActionUrlResolver
        arguments:
            - '@router'
            - '@oro_api.rest.routes_registry'
            - '@oro_api.value_normalizer'
            - Oro\Bundle\CMSBundle\Entity\Page
        tags:
            - { name: oro_frontend.api.resource_api_url_resolver, routeName: oro_cms_frontend_page_view, requestType: rest }
```

Here are some useful implementations of <a href="https://github.com/oroinc/customer-portal/blob/master/src/Oro/Bundle/FrontendBundle/Api/ResourceTypeResolverInterface.php" target="_blank">ResourceTypeResolverInterface</a> and <a href="https://github.com/oroinc/customer-portal/blob/master/src/Oro/Bundle/FrontendBundle/Api/ResourceApiUrlResolverInterface.php" target="_blank">ResourceApiUrlResolverInterface</a>:

- <a href="https://github.com/oroinc/customer-portal/blob/master/src/Oro/Bundle/FrontendBundle/Api/ResourceTypeResolver.php" target="_blank">ResourceTypeResolver</a> - resolves a resource type by a route name.
- <a href="https://github.com/oroinc/customer-portal/blob/master/src/Oro/Bundle/FrontendBundle/Api/ResourceRestApiGetActionUrlResolver.php" target="_blank">ResourceRestApiGetActionUrlResolver</a> - resolves the URL of the [get](actions.md#get-action) action REST API resource by a route name.
- <a href="https://github.com/oroinc/customer-portal/blob/master/src/Oro/Bundle/FrontendBundle/Api/ResourceRestApiGetListActionUrlResolver.php" target="_blank">ResourceRestApiGetListActionUrlResolver</a> - resolves the URL of the [get_list](actions.md#get-list-action) action REST API resource by a route name.

<!-- Frontend -->
