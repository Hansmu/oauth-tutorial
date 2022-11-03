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

**Application flow**

For public apps like Facebook, registering your app is pretty straightforward. Otherwise, it's not quite as self-serve.
When you register an app, you get a client_id (public information), and you may or may not also get a client_secret
(secret). You'll also have to enter a bunch of different information, but at a minimum you'll be able to enter a 
name and a redirect URI or two. A wildcard URI should not be allowed, as that's a potential attack vector.

1. The user turns to the app
2. The app generates a random secret. This is unique for each flow execution. This is called the PKCE code verifier.
It holds the random secret in memory. Also, a hash is generated from it called a PKCE code challenge.
   ![Code verifier secret](./images/code_verifier_secret.png)
   ![Code challenge](./images/code_challenge.png)
3. The app redirects the user to the authorization server and adds the hash to the call. So this is happening in the
front channel because it's through the browser. A call to the auth endpoint follows, to which you add a bunch of query
string parameters: 
   * response_type=code, this tells the server that you're doing the authorization code flow.
   * client_id=CLIENT_ID, this tells the server which app is making the call.
   * redirect_uri=REDIRECT_URI, this has to match one of the URIs that you have registered for your app.
   * scope=SCOPE_VALUE, this is for the scopes that you are trying to access.
   * state=XXXX, this was originally used for CSRF protection, but PKCE fixes the same issue. So this could be 
   used to save application specific data. Although, this is only safe to be used this way if you are sure that
   the OAuth server supports PKCE. If not, then this has to be a random value.
   * code_challenge=XXXXXXXXXXXXXX, this is the hash that was generated initially.
   * code_challenge_method=S256, this is the hash method that was used to produce the hash.
     ![Redirect URL](./images/auth_redirect_url.png)

   The user does all the auth that is necessary on the authorization server.

4. The authentication server returns a temporary code for the app to use. The browser receives the code and the code
is forwarded to the backend. There is also the possibility that some error will have occurred.
   ![Auth server redirect back](./images/auth_server_redirect_back_to_app.png)
   ![Error redirect](./images/error_redirect.png)
5. The server should validate that the state value matched, if no PKCE is available. The server forwards the code,
the plaintext PKCE code secret that was initially created, and asks for a token.
   * grant_type=authorization_code, tells the server that you're doing an authorization code grant.
   * code=AUTH_CODE_HERE, the authorization code that was provided by the authorization server back to the app.
   * redirect_uri=REDIRECT_URI, the URI that was used in the request.
   * code_verifier=VERIFIER_STRING, the plaintext code that was generated at the start.
   * client_id=CLIENT_ID, the app identifier for the authorization server.
   * client_secret=CLIENT_SECRET, the secret of the app.
     ![Token post](./images/token_post.png)
6. The server verifies the secret and then returns an access token. Possibly with a refresh token.
   ![Token response](./images/token_response.png)
   If the current access token expires, then a refresh token could be used to get a new access token without doing 
the entire flow again. This then returns a new access token, and possibly a new refresh token. There are a whole bunch
of reasons why a refresh token might fail, though, so it's entirely possible for this call to fail. So when it fails, 
then a full new authorization flow should be attempted.
   ![Refresh token call](./images/refresh_token_call.png)

PKCE was initially developed for mobile apps. The guidance is now recommended for all kinds of applications. With
server side apps there is still a subtle attack you can pull off called an authorization code injection attack, where 
you can swap authorization codes and end up logged into as someone else. So PKCE helps against this. You may find 
servers that do not support PKCE for confidential clients. In that case you should recommend that they add PKCE support.
You can still add PKCE parameters, even if they aren't support today, as the server simply ignores parameters it doesn't
care about, but when PKCE support becomes available, then it'll become so automatically in the future.

![Auth code flow](./images/auth_code_flow.png)

**Native apps flow**

Native apps lose a lot of the built-in security that exist in browsers. For example, it checks the DNS, validates HTTPS.
The redirect URL is where the differences come in. The native app opens up an in-app browser to the auth server, and then
it returns a redirect URL. 

There are two ways how a redirect URL can happen within a mobile app. 

It used to be common
that applications would be able to just define a custom URL scheme and any time anything in the phone launched a URL 
matching that scheme, that app would launch instead. The issue is that there's no registry of these, so any app can
claim any URL scheme. If someone figures out your app scheme, then they can just create their own app with the same
scheme, and they'd race for the launch. Opens up a chance to steal auth codes in the redirect.

The more modern way for redirects to be claimed is by claiming URL patterns. Also known as deep linking. This is where
your mobile app can claim a URL pattern, including a full domain name or even path. And then again, once that's 
registered, any app that launches your URL will launch your app instead. This is how you get things like when you
click on a link to Amazon, you get taken to the Amazon app with that product loaded or linking to an Instagram photo
where you get dropped right into the Instagram app instead of the website. What makes this more secure is that the
app developer has to actually prove that they control that domain name. There are, however, still ways it can fail
so we can't trust it as much as the browser. And that's really why PKCE is so important.

One of the most important parts about the OAuth flow in a mobile environment is how the mobile app launches 
the browser to the OAuth server to start the flow. 

It used to be that the most secure way to do it was to
launch the native app (Safari, Chrome). You launch the browser, your app goes into the background, they'd log in,
switch back. It's more fragile since the user leaves the context of the app. Another thing that was previously used
was embedding a webview into the app. A couple of issues with that is that the user could not see the address bar,
can't verify where they are, might be on an attacker site. Also, cookies aren't shared with the system. So if you're
logged in the browser, then you wouldn't be in the webview. Additionally, the app has full control over the webview,
so they could read the password if they wanted to.

Things have got a lot better now. We now have APIs which are able to launch a browser in a secure way within the 
application, which means the user never actually leaves the app. The user is inside the app, they click a button, 
the web view pops up in front of the application, they can log in there, and then it goes away, and they're still
within the application. The nice thing about this is that the user never leaves the app. Also, the app doesn't
have access to the browser. Also, it shares cookies with the system, thus saving the user from having to log in 
again.

**Authorization code flow for native apps**

The flow is pretty much the same as for the regular flow, but you don't move around a client secret key.

![Auth code flow for native apps](./images/native_apps_code_flow.png)

Depending on the server configuration, the server might return a refresh token or the app might need to
explicitly ask for it. Since there is no client secret, then anyone that possesses the refresh token would 
be able to get a new token. So you need to take proper care to protect the refresh token. Luckily enough, most
of this is already taken care of by the native app platforms, isolating apps from each other and things like that.
There's also a particularly useful API on mobile to better keep your refresh token safe. The secure storage API
is one which the app's code can't access anymore with biometric validation after storing into it. 
When trying to log in again with the  refresh token, the app prompts the user for biometrics to be able to 
access it, making the refresh token available to the app. It then makes that request using the refresh token, and it
gets a new access token back, and the user is logged in. The user didn't have to see a web browser, just looks like
a seamless  FaceID or thumbprint authentication. Your app wouldn't even be able to accidentally extract the refresh
token.

**OAuth for SPAs**

SPAs have several limitations. 
* There is no way of shipping credentials in the app, so they are considered public clients in OAuth terms. This can be
mitigated with PKCE, though.
* Browsers are vulnerable to a whole bunch of attacks. Ex. XSS. XSS can come from your app or from third party scripts that
you have included in your app. Users might also have custom plugins that can cause
security issues. Browsers also have no secure storage. If your code can read the storage, then there's a chance an
attacker's JS can as well.

OAuth servers can have significantly different policies for single-page apps compared to other kinds of apps. 
For example, refresh tokens might be disabled completely or they might be single use only. Token lifetimes might be
shorter to reduce the risk from leaking.

Similar to a native app, you won't have a client secret. The generated secret is usually stored in LocalStorage or
SessionStorage.

![SPA auth flow](./images/spa_auth_flow.png)

There are a couple of ways to secure a token in a browser
* Keeping it in memory will make it a lot harder for an attack to get to it. This means that it won't persist through
refreshes and tabs, though.
* Storing it in a Service Worker. Service Workers are completely isolated from the main browser window, so XSS
attackers won't actually have access to it. The downside is that it's more complicated and doesn't work in IE11.
You'd have to use a service worker as a little isolated OAUth client, meaning that your JS can't make API calls itself
anymore and needs to instead tell the service worker to make the API call, and then it'll just handle the response.
An XSS attacker could still tell your service worker to go make those requests.
* A WebCrypto API could also be used, which isn't supported in IE11. It generates a private key, while preventing that
private key from being extracted. You can use the key to encrypt things that you might want to store. Better than
storing in plaintext into LocalStorage, but an attacker could still use that key to decrypt the storage. But it
would be a more targeted attack than a LocalStorage look around.

So the only way to keep tokens safe from JS based attacks is to not give JS access to tokens at all. This can be 
achieved by using a HTTPOnly session cookie to the browser. Although, this only works if you have a backend.

![HTTP Session Cookie Flow](./images/http_session_cookie_flow.png)