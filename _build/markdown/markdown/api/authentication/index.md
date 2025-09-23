<a id="web-services-api-authentication"></a>

# Authentication

A RESTful API should be stateless. This means that request authentication should not depend on cookies or sessions.
Instead, each request should come with some authentication credentials.

Out-of-the-box, OroPlatform provides the following authentication mechanism:

* [OAuth Authentication](oauth.md)
