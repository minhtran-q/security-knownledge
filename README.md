# Security Knownledge
## Oauth2

### Authorization code

<details>
  <summary>How dose it work?</summary>
  <br/>
  
  ![](images/oauth_web_server_flow.png)
  
  + Ref: https://docs.oracle.com/cd/E50612_01/doc.11122/oauth_guide/content/oauth_flows.html
  + Ref: https://portswigger.net/web-security/oauth/grant-types
  
</details>
<details>
  <summary>Detail flow</summary>
  <br/>
  
  1. Redirect the user to the authorization endpoint with the following parameters:
  
  | Parameter  | Description |
  | ------------- | ------------- |
  | response_type  | Required. Must be set to code.  |
  | client_id  | Required. The Client ID generated when the application was registered in Identity Server  |
  | redirect_uri  | Where the authorization code will be sent. This value must match one of the values provided in Identity Server  |
  | scope  | Optional. A space delimited list of scopes, which indicate the access to the Resource Owner's data being requested by the application.  |
  | state  | Optional. Any state the consumer wants reflected back to it after approval during the callback.  |
  
  ```
  https://apigateway/oauth/authorize?client_id=SampleConfidentialApp&
  response_type=code&&redirect_uri=http%3A%2F%2Flocalhost%3A8090%2Fauth%2Fredirect.html&
  scope=https%3A%2F%2Flocalhost%3A8090%2Fauth%2Fuserinfo.email
  ```
  _URL example_
  
  2. The response to the above request is sent to the redirect_uri. If the user approves the access request, the response contains an authorization code and the state parameter (if included in the request). If the user does not approve the request, the response contains an error message. All responses are returned to the Web server on the query string. For example:

`https://localhost/oauth_callback&code=9srN6sqmjrvG5bWvNB42PCGju0TFVV`

  3. After the Web server receives the authorization code, it may exchange the authorization code for an access token and a refresh token. This request is an **HTTPS POST**
  
  | Parameter  | Description |
  | ------------- | ------------- |
  | grant_type  | Required. Must be set to authorization_code. |
  | code  | Required. The authorization code received in the redirect above. |
  | redirect_uri  | Required. The redirect URL registered for the application (back-end client).  |
  | client_id*  | Optional. The client_id obtained during application registration. |
  | client_secret*  | Optional. The client_secret obtained during application registration. |
  
  _* If the client_id and client_secret are not provided as parameters in the HTTP POST, they must be provided in the HTTP Basic Authentication header (Authorization base64Encoded(client_id:client_secret))._
  
  ```
  POST /api/oauth/token HTTP/1.1 
  Content-Type: application/x-www-form-urlencoded 

  client_id=SampleConfidentialApp&client_secret=6808d4b6-ef09-4b0d-8f28-3b05da9c48ec
   &code=9srN6sqmjrvG5bWvNB42PCGju0TFVV&redirect_uri=http%3A%2F%2Flocalhost%3A809
   0%2Fauth%2Fredirect.html&grant_type=authorization_code&format=query
  ```
  4. After the request is verified, the Authentication Server sends a response to the client.
  
  | Parameter  | Description |
  | ------------- | ------------- |
  | access_token  | The token that can be sent to the Resource Server to access the protected resources of the Resource Owner (user). |
  | refresh_token  | A token that may be used to obtain a new access token. |
  | expires  | The remaining lifetime on the access token.  |
  | type  | Indicates the type of token returned. At this time, this field always has a value of **Bearer**. |
  
  ```
  HTTP/1.1 200 OK
  Cache-Control: no-store
  Content-Type: application/json
  Pragma: no-cache{
      "access_token": “O91G451HZ0V83opz6udiSEjchPynd2Ss9......",
      "token_type": "Bearer",
      "expires_in": "3600",
  }
  ```
  
  5. After the Web server has obtained an access token, it can gain access to protected resources on the Resource Server by placing it in an Authorization: Bearer HTTP header
  
  ```
  GET /oauth/protected HTTP/1.1
  Authorization: Bearer O91G451HZ0V83opz6udiSEjchPynd2Ss9
  Host: apigateway.com
  ```
  or in curl
  
  `curl -H "Authorization: Bearer O91G451HZ0V83opz6udiSEjchPynd2Ss9" https://apigateway.com/oauth/protected`
</details>
<details>
  <summary>What is the purpose of authorization code in OAuth?</summary>
  <br/>
  
  It's possible to do it with a single request - it's called the _**implicit flow**_ then.
  
  The general idea of using access code (authorization flow) instead of directly returning the _**tokens**_ and _**client secrect**_ is to **hide** them from the end user. The second request is done usually by the backend server instead of a browser.
  
  This exchange of _authorization code_ doesn't involve the user’s browser so there is no way access tokens are stored in history of the browser.

  + Ref: https://stackoverflow.com/questions/53995441/what-is-the-purpose-of-authorization-code-in-oauth
  + Ref: https://stackoverflow.com/questions/7522831/what-is-the-purpose-of-the-implicit-grant-authorization-type-in-oauth-2
  + Ref: https://www.quora.com/Why-does-OAuth-server-return-a-authorization-code-instead-of-access-token-in-the-first-step
  
</details>

### Client credential
### Resource owner password credential
### Implicit
### OAuth 2.0 Grant Types With Their Use Cases

<details>
  <summary>Explain</summary>
  <br/>

  
  + Ref: https://www.intelegencia.com/blog/technology/oauth-2-0-grant-types-with-their-use-cases#:~:text=The%20best%20use%20case%20for,the%20app's%20credential%20get%20validated.
  
</details>

## OpenId Connect
### OpenId Connect vs Oauth2

<details>
  <summary>Brief defination about OpenID Connect and OAuth2</summary>
  <br/>
  
  **OAuth2:** This is a standardized set of rules that defines how applications can access resources on a server on behalf of a user. It focuses on authorization and token management.

  **OpenID Connect:** This is an identity layer that sits on top of the OAuth2 protocol by adding mechanisms for user authentication. It uses OAuth 2.0 flows (like the Authorization Code Flow) to obtain tokens but defines additional features like ID Token, Discovery, Standardized Scopes.
  
</details>
<details>
  <summary>Access token vs Id Token</summary>
  <br/>
  
</details>

## SSL/TLS
## JWT
### What is JSON Web Token?

### How does it work?
JSON Web Token (JWT) is an open standard _(RFC 7519)_ that defines a way for securely transmitting information between parties as a JSON object. 

<details>
  <summary>Overview</summary>
  <br/>
  
  ![](images/signing_overview.png)
  
  + Ref: https://cryptobook.nakov.com/digital-signatures
  
</details>
<details>
  <summary>Explanation</summary>
  <br/>
  
  ![](images/how_signing_work.png)
  
  Typically the input message is **hashed** and then the **signature** is calculated by the signing algorithm. Most signature algorithms calculate with the message hash + the signing key (**private key**)
  ```
  signMsg(msg, privKey) 🡒 signature
  ```
  
  ![](images/jwt_signing.png)
  
  Message signatures are verified by the corresponding verification key (**public key**). So, to validate a digital signature, the recipient

  + Calculates a hash of the same data (file, message, etc.),
  + Decrypts the digital signature using the sender's PUBLIC key, and
  + Compares the 2 hash values.
  ```
  verifyMsgSignature(msg, signature, pubKey) 🡒 valid / invalid
  ```
  
  + Ref: https://cryptobook.nakov.com/digital-signatures
  + Ref: https://stackoverflow.com/questions/18257185/how-does-a-public-key-verify-a-signature
  
</details>
<details>
  <summary>JWT in Client-Server</summary>
  <br/>
  
  ![](images/client-server-jwt.png)
  
  + Ref: https://cryptobook.nakov.com/digital-signatures
  
</details>
<details>
  <summary>Why not have the public key in the JWT payload for convenience?</summary>
  <br/>
  
  + Ref: https://www.google.com/search?q=dich&rlz=1C1GCEU_enVN945VN945&oq=dich&aqs=chrome.0.69i59j0i512j0i131i433i512j0i3j0i131i433i512j69i60l3.534j0j7&sourceid=chrome&ie=UTF-8
  
</details>

### Encryption vs Signing. What’s the difference?
<details>
  <summary>Encryption</summary>
  <br/>
  
  + Ref: https://www.encryptionconsulting.com/education-center/encryption-and-signing/#:~:text=Encryption%20uses%20a%20key%20to,of%20encryption%20in%20its%20process.
  
</details>
<details>
  <summary>Signing</summary>
  <br/>
  
  + Ref: https://www.encryptionconsulting.com/education-center/encryption-and-signing/#:~:text=Encryption%20uses%20a%20key%20to,of%20encryption%20in%20its%20process.
  
</details>

## REST

REST (representational state transfer) is a software architectural style that was created to guide the design and development 

<details>
  <summary>What is a Stateless REST API?</summary>
  <br/>

   Stateless REST APIs do not establish or maintain client sessions. Clients are responsible for providing all necessary information in each request, such as authentication tokens, credentials, or context data. The server does not store client-specific session data.
  
</details>

<details>
  <summary>Core Principles of REST</summary>
  <br/>

  + Client/Server - Client are separated from servers by a well-defined interface.
  + Stateless - each request from the client to the server must contain all of the information necessary to establish and complete a request.
  + Cacheability - 
  + Layered System -
  + Uniform Interface - The uniform interface includes using standard HTTP verbs (GET, POST, PUT, DELETE, etc.), standard HTTP error responses, and resource identification through URI.
  + Code on Demand (optional) -
  
</details>


