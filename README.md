# oauth2
[OAuth2 WorkFlows](https://auth0.com/docs/get-started/authentication-and-authorization-flow/which-oauth-2-0-flow-should-i-use)
[The Nuts and Bolts of OAuth 2.0 Exersizes and the course](https://oauth.school/)

### Authorization code flow
[Authorization code flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow)
[Authorization Code Flow with Proof Key for Code Exchange (PKCE)](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow-with-pkce)

### Device Authorization Flow
[Device Authorization Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/device-authorization-flow)
### User Credentials flow
[Client Credentials Flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow)

Is one of the most simplest flow, because it's for Machine-to-Machine communication. There is no user interaction.It involves an application exchanging its application credentials, 
such as client ID and client secret, for an access token.
Once you register the app, you should see a `client ID` and `client secret`, and **that's allyou'll need. In order to get an access token, you'll be making a POST request to the token endpoint** 
with the following parameters: grant_type=client_credentials.If you're requesting a down-scoped token, then you'll also include scope with whatever scope you're requesting. The only thing to look 
out for now is how to include the client credentials in the request. The authorization server may either want the client credentials in the post body or in an HTTP header as Basic Auth, and this is
going to depend on the server you're working with. So you'll want to double check the documentation of that server. Some servers might also support both, but let's say, for example, the server expects
the credentials in the POST body.


### Access Token vs ID Token
[Access Token vs ID Token](https://auth0.com/blog/id-token-access-token-what-is-the-difference/)

The tokens look very similar, but they are used for different purposes. 
**Access Token** is like a key for a hotel room. We do not really care what is inside it, we just use it to open a door. Applications are going to get an access token from the authorization server, hold on to it and then use it at the API.
On the other hand **ID Token** is like a user passport. Applications are going to get an ID token from the authorization server and unpack it and learn about the user.

!!! Now it's also important to remember that not every OAuth server uses JSON Web Tokens for access tokens. It's entirely possible that access tokens might be something completely different. 
And that's why the application should make no assumptions about what format access tokens are in. But ID tokens are always JSON Web Tokens and applications have to know how to validate them in order to use them.


### How Application Gets ID Token (OpenID Connection)
[OpenID scopes](https://auth0.com/docs/get-started/apis/scopes/openid-connect-scopes)

So you can see it's very straightforward to just get an ID token along with the access token in this way. One of the really nice things about doing it this way is that because byou obtained the ID token 
over the back channel by directly exchanging that authorization code for the ID token, you actually can forget that it's a JSON Web Token and you can skip all the verifications, because you already 
know where it came from. So you don't need to validate the signature and you already know that it's valid so you don't need to check the expiration dates. You can basically just extract the parts in 
the middle that you care about and just use the data directly. So this is my preferred way because this is just vastly simplifies.

Remember that an ID token is Base64-encoded JSON data that's signed. Now, the signature is important if you get the ID token from another untrusted source.
But if you get it over this trusted connection, over the back channel, over HTTPS, then you don't need to bother checking the signature because you know it can't be tamperedwith. 

Now, there's a different way you can get an ID token, and that's to use a different flow. If you actually use instead of `response_type=code`, if you use `response_type=id_token`, that's telling the authorization server that you actually don't want an access token at all. 
And that will give you just an ID token. And in that mode, response_type=id_token, that actually returns the ID token in the redirect instead of the authorization code.
And that looks a lot like the Implicit flow in OAuth, where it returns the token without the intermediate authorization code.


### Access Token Types and their Tradeoffs
So at a high level, there are essentially two different families have access tokens. Access tokens can either be a 
- **reference token**, which is essentially just a long, random string of characters, that doesn't actually mean anything, it's a pointer to something.
- Or the access token could be **structured** or **self-contained**. And that's the case where the access token string itself actually contains data in some sort of format. Now, in both cases, there's many different options you have to actually
implement this. For example, reference tokens could be implemented in a relational database where you have a table for tokens. One column is the actual token string itself and the other columns are things like the user ID or the lifetime of the token or the scopes issued to it.
However, you could also implement reference tokens using a caching layer like Memcache or Redis where you can generate a random string that becomes the cache key.
Then you store any data about the token inside of that database. ⚠️**The overall idea with `reference tokens` is that the token string itself doesn't actually mean anything itself.**

**reference tokens Pros and Cons:**

✅Pros:
- Simple
- Easy to revoke
- Token data is not visible (it's just a random string)

❌ Cons:
- Token should be stored somewere. It's not a big deal, but we should think about that especially if the data store should scale 
- Requires network to validate the token. It becoms more important on scale when the storage starting to gets more and more requests.
- Sometimes it requires not only a storage but also an additional API that's used by services to request stored tokens.


**Self-encoded (contains meaningful data) tokens Pros and Cons:**

✅ Pros:
Services which get the token know how to validate it and read the data inside the token. So where this really starts to shine is if you're using an OAuth server that's not part of your API code base.
If you're building out an API and using some sort of other product as an OAuth server, then it makes a lot of sense to not have to have any sort of shared storage between those two systems since they operate independently. For example, if you're using Okta as your OAuth server, then Okta is the one creating these access tokens. Your APIs have nothing to do with the code base behind Okta.
You just know how to validate JSON Web Tokens and then you can build your APIs, which are validating access tokens just by looking at the string and never need to go back and ask over the network whether the token is valid.

And that's why you do see self-encoded tokens, especially JSON Web Tokens, in so many authorization servers. So it's really the best option if you're building a larger scale API, but especially if you're using an OAuth server that is completely separate from your APIs. And that's essentially what you're doing when you pick up a product off the shelf. Whether it's an open source implementation of an OAuth server or a service. Using self-encoded access tokens is a lot more scalable and gives you a lot of advantages.

- Do not need shared storage
- Can be validated without network.
  

❌ Cons:

So anything you pack into the access token, whether that's the user's ID or email or nameor groups or things about the user, all that's in the token and it's just base64 encoded.
Like if you look at the string, it doesn't look like it says anything, but it's really just base64 encoded data. So you can just copy this middle part here, run it through a base64 decoder and you'll see a JSON blob. And the trick is that anybody can do that.

- JWT Tokens are visible (can be decoded and they contain some information)
- No way to revoke them before they are expired
