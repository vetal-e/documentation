<a id="web-api-security"></a>

# Configure Stateless Security Firewalls

The Symfony framework allows for the creation of stateless firewalls. In this case, the security token is not serialized for a session.

However, when API calls are utilized in AJAX requests from the UI, the user’s token data from the current session must be used instead of the firewall credentials. For this purpose, the firewall should have the <a href="https://symfony.com/doc/6.4/reference/configuration/security.html#firewall-context" target="_blank">context</a> parameter with the context name that the system can use to authenticate the user.

For example:

```yaml
security:
    firewalls:
        some_stateless_firewall_with_AJAX_requests:
            stateless: true
            context:   main
            # ...
```

<!-- Frontend -->
