<h1>OAuth</h1>

Single sign-on (SSO) is a centralized session and user authentication service in which one set of login credentials can 
be used to access multiple applications. Its beauty is in its simplicity; the service authenticates you on one 
designated platform, enabling you to then use a variety of services without having to log in and out each time.

In the most common arrangement, the identity provider and service provider establish a trust relationship by exchanging 
digital certificates and metadata, and communicate with one another via open standards such as 
Security Assertion Markup Language (SAML), OAuth, or OpenID.

There are three key terms you need to know in SSO lingo:
* Service provider: In an SSO context, this is an application or website that a user might want to log into—anything 
from an email client to a bank website to a network share. Most platforms like these would include their own 
functionality for authenticating users if they were standing alone, but that's not the case with SSO.
* Identity provider: With SSO, responsibility for authenticating users is fielded out to an identity provider—generally, 
the SSO platform itself. When the user attempts to access the service provider, the service provider will consult with 
the identity provider to ensure that the user has proven that they are who they claim to be. The service provider can 
put parameters around how authentication works: for instance, it can require that the identity provider use two-factor 
authentication (2FA) or biometrics. The identity provider will either ask the user to log in, or, if they've logged in 
recently, may simply let the service provider know that without troubling the user further.
* Tokens: These are small collections of structured information that are digitally signed to ensure mutual trust, and 
they're the medium by which the service and identity providers communicate. It's through these tokens that the identity 
provider will tell the service provider that the user has authenticated—but, crucially, the tokens don't include 
authentication data like the user's password or biometric data. As a result, even if the tokens are intercepted by an 
attacker or the service provider's systems are breached, the user's password and identity remain secure. The user can 
also use the same login credentials for any service providers using that identity provider.

OAuth is an open-standard authorization protocol or framework that describes how unrelated servers and services can 
safely allow authenticated access to their assets without actually sharing the initial, related, single logon credential. 
In authentication parlance, this is known as secure, third-party, user-agent, delegated authorization.

**Roles**
* **The user (Resource Owner)** - the person with the account. _For example, I’m the Resource Owner of my Facebook profile._
* **The device (User Agent)** - the device being used by the user
* **The application (The OAuth Client)** - the device is running an application or accessing the 
application. _Ex. a client application could be a game requesting access to a users Facebook account._
* **The API (Resource Server)** - what the application is making requests to. Can be one or multiple
servers. _For instance, Facebook is a resource server (or has a resource server)._
* **The authorization server** - manage access to the server that it's protecting. The only one who
will see the user's password. Exchanges a cookie/token for the password. In smaller applications
the authorization server could be combined with the resource server. The authorization server verifies the identity of 
the user then issues access tokens to the application.

**Application types**
* Confidential - has the ability to be deployed with a client secret, the secret won't be visible
to anyone using the app. So things that end up on a device that is not touchable by the client.
Identity has been confirmed.
* Credentialed - a client which has credentials, but its identity isn't established by the server.
Dynamic registration is used to get a client secret. So an initial query is being sent to register
a client, but the initial query could be sent by anyone. That initial query would contain no
identification information. Can conclude that all the following requests are made by the same 
client instance, though. Can approach the client as a half-and-half. Know that refresh tokens
are all being requested by the same client, but give out the lifetime of the token as if to
a public client.
* Public - code that gets run on a client's device. Secrets will be accessible by the user. For
example SPA or mobile apps. Identity has not been confirmed.

The distinction matters, because the auth server might behave differently depending on who is
making the request. You might skip a step if you can guarantee that the application is not being
mimicked. So with a confidential app, you could be sure of it, as the secret key should never be
visible to the client, thus could not be re-used for faking requests. Skipping the consent screen,
including refresh tokens, limiting the token lifetime, these are some things that the server
might choose to do differently based on the application type.

User consent form - user consent represents a user's explicit permission to allow an application to access 
resources protected by scopes. Consent grants are different from tokens because a consent can outlast a token, 
and there can be multiple tokens with varying sets of scopes derived from a single consent. When an application 
needs to get a new access token from an authorization server, the user isn't prompted for consent if they have 
already consented to the specified scopes. Consent grants remain valid until the user or admin manually revokes 
them, or until the user, application, authorization server, or scope is deactivated or deleted.

Without the user consent form, the flow would have to utilize the password grant, which means handing your password
over to a third party app, which raises questions of trust. Image going to a random website and being asked for your
Facebook password. Also, there would be no limiting the scope of what the
app can use. The password grant also eliminates knowing if the user is at the computer. The password might just be
saved in the app. The consent form happens on the authorization server, where the user would be asked for the password,
thus verifying that the user is at the computer. If everything is happening through the authorization server, then it's
pretty easy to add MFA as well. First party apps might want to skip the consent form, as there isn't a trust issue. Still
has to do the redirects, though.

**How data moves around**

Back channel - a direct communication between the client and the token endpoint over HTTPS. We know who we're talking
to because of the certificates. Data is encrypted in transit. Can trust the response because we know where it came from.
This is akin to hand-delivering a package face to face. This is the secure method and we should prefer this whenever
possible. Back channel IS NOT a backend. A JS app can absolutely use the back channel. What this means is JS making
the request from JS code, like an AJX or a fetch request. Making the request from JS has the same properties as making
a request from a backend server. That's because you get certificate verification and an encrypted connection. This is
very different from sending data in the front channel, which is just using the address bar to move data where there are
a lot more failure points, and chances for data to be intercepted.

Front channel - indirect communication between the client and the authorization endpoint via the user's agent 
(using the browser's address bar). Akin to handing a package over a delivery service. You hand the package over and
the service delivers it. There's no direct link between the sender and the receiver. You can never be sure that the
delivery service doesn't do something you don't expect. Neither the sender nor receiver can be sure that what's being
sent/received is untampered. With the back channel, we have no way of asking for a consent from the user. This is why
we use the front channel, because we can insert the user into the flow. 
The flow would look something like this:
* First the application server would send a request to the authorization server, saying what it wants to do. This
includes the application's identifier, scopes etc. It's usually safe to send this in the front channel, as nothing
in this request is particularly sensitive.
* The user logs in and authorizes the request. The authorization server is ready to send an access token back to the
application and redirect the user back to the application. The access token cannot be sent via the redirect, because
that would be like sending an access token via the mail. The authorization server would have no guarantees that it was 
actually delivered back to the application, and the application wouldn't have any guarantees that the access token is
really from the authorization server. This approach is described in the core OAuth spec, but it's not recommended because
of the lack of security. This is called the implicit flow, and it uses the front channel for all communication. It's
included in the spec, because it used to be that browsers had no other option, as CORS requests used to be not possible. 
The solution is to deliver the access token in the back channel instead. 

**Application identity**

Each application has its own identifier called a client ID, which it uses to identify itself throughout the OAuth flow.
The redirect link to the authorization server is sent with a ClientID in it, among all the other important parameters.
The authorization server send an authorization code back to the server. This is basically like a short-lived coupon that
can be used to redeem an access token. Since this is done through the front channel, then the server cannot be sure that
whoever is redeeming it is, indeed, the client server. The client secret is used to verify the identity of the client
server. So in order for the client server to successfully redeem the authorization code, it needs to provide that and
a correct client secret. However, SPAs and mobile apps cannot be deployed with a client secret. They use PKCE.

The Redirect URI is where the user will be sent to from the authorization server after they log in. The authorization
code will be sent there. It might be a URL or a custom URL scheme. The HTTPS URLs are considered to be globally unique,
as you own that URL. Custom URL schemes aren't usable as application identity, as there is no guarantee of uniqueness
and ownership. So an application's HTTPS URL is part of an application's identity. With SPAs and mobile apps we can 
use this for their identity. You have to consider security policies based on the context.