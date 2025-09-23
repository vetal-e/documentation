<a id="web-services-api-authentication-oauth"></a>

<a id="index-0"></a>

# OAuth Authentication

<a href="https://oauth.net/2/" target="_blank">OAuth 2.0</a> is the industry-standard protocol for authorization. OAuth 2.0 focuses on client developer simplicity
while providing specific authorization flows for web applications, desktop applications, mobile phones,
and living room devices.

It is implemented by the OroOAuth2ServerBundle that supports
<a href="https://oauth.net/2/grant-types/authorization-code/" target="_blank">OAuth 2.0 Authorization Code Grant</a> (with PKCE extension), <a href="https://oauth.net/2/grant-types/client-credentials/" target="_blank">OAuth 2.0 Client Credentials Grant</a> and <a href="https://oauth.net/2/grant-types/password/" target="_blank">OAuth 2.0 Password Grant</a>.

For more details on managing these accesses and how to generate credentials in the Oro application, see Manage OAuth Applications
and Manage Customer User OAuth Applications.

## Generate Tokens

* [Authorization Code Grant Type](oauth-authorization-code.md)
* [Client Credentials Grant Type: Generate Token](oauth-client-credentials.md)
* [Password Grant Type: Generate Token](oauth-password.md)
* [Password Grant Type: Refresh Token](oauth-password-refresh.md)

#### NOTE
In order to use OAuth authentication, private and public keys should be generated and placed
to the server. Please contact your administrator or please follow
the OroOAuth2ServerBundle documentation
if you see the following error message:

*The encryption key does not exist.*

#### NOTE
If the system has the customer portal package installed, OAuth authorization for customer users
to the storefront API resources is enabled automatically.

<!-- Frontend -->
