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
### Fundamental concepts
<details>
  <summary>What is JWT</summary>
  <br/>

  JSON Web Token (JWT) is an open standard _(RFC 7519)_ that defines a way for securely transmitting information between parties as a JSON object. 

  **Structure of JWT:**

  + Header: This part typically consists of two elements
    + The type of the token, which is JWT.
    + The signing algorithm being used, such as HMAC SHA256 or RSA.
  ```
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

  + Payload: This part contains the claims. Claims are statements about an entity (typically, the user) and additional data. There are three types of claims:
    + Registered claims: Predefined claims like `iss` (issuer), `exp` (expiration time), `sub` (subject), and `aud` (audience).
    + Public claims: Custom claims that can be defined by users.
    + Private claims: Custom claims that are shared between parties that agree on using them.
  ```
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

  + Signature: This part is used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn’t changed along the way.
  ```
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret
  )
  ```
</details>
<details>
  <summary>JWT Flow</summary>
  <br/>
  
  ![](images/client-server-jwt.png)

  **1. User Authentication:** The user logs in with their credentials (e.g., username and password). Then the server verifies the credentials.
  **2. Token Creation:** The JWT is signed using a secret key or a public/private key pair.
  **3. Token Transmission:** The server sends the JWT back to the client (usually in the response body or as a cookie). Then the client stores the JWT, often in local storage or a cookie.
  **4. Subsequent Requests:** The client includes the JWT in the HTTP headers (usually in the Authorization header with the Bearer schema)
  **5. Token Verification:** The server verifies the JWT by checking its signature and ensuring it has not expired. If the token is valid, the server processes the request and sends the appropriate response.
  
  + Ref: https://cryptobook.nakov.com/digital-signatures
  
</details>
<details>
  <summary>Signing & Verification in JWT</summary>
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

### Other questions
<details>
  <summary>Private & public key</summary>
  <br/>
  
  ![](images/signing_overview.png)
  
  + Ref: https://cryptobook.nakov.com/digital-signatures
  
</details>
<details>
  <summary>Encryption vs Signing</summary>
  <br/>
  
  + Ref: https://www.encryptionconsulting.com/education-center/encryption-and-signing/#:~:text=Encryption%20uses%20a%20key%20to,of%20encryption%20in%20its%20process.
  + Ref: https://www.encryptionconsulting.com/education-center/encryption-and-signing/#:~:text=Encryption%20uses%20a%20key%20to,of%20encryption%20in%20its%20process.
</details>
<details>
  <summary>Why not have the public key in the JWT payload for convenience?</summary>
  <br/>
  
  + Ref: https://www.google.com/search?q=dich&rlz=1C1GCEU_enVN945VN945&oq=dich&aqs=chrome.0.69i59j0i512j0i131i433i512j0i3j0i131i433i512j69i60l3.534j0j7&sourceid=chrome&ie=UTF-8
  
</details>

## CORS

<details>
  <summary>Same Origin Policy</summary>
  <br/>
  
  Two URLs are considered to have the same origin if they share the following three components:

  + Protocol: The communication method used (e.g., http, https).
  + Domain: The domain name (e.g., example.com).
  + Port: The port number (if specified).

  Same Origin:
  + `http://www.example.com/path1` and `http://www.example.com/path2`
  + `https://example.com:8080/foo` and `https://example.com:8080/bar`
  
  Different Origin:
  + `http://www.example.com` and `https://www.example.com` (different protocol)
  + `http://www.example.com` and `http://www.anothersite.com` (different domain)
  + `http://www.example.com:80` and `http://www.example.com:8080` (different port)

  _Note:_ Different subdomains are considered different origins (e.g., `www.example.com` and `blog.example.com`).
</details>
<details>
  <summary>Without Same Origin Policy</summary>
  <br/>
  Imagine you are logged into your online banking site (https://bank.com). At the same time, you visit another website (http://malicious.com). Without the same-origin policy, it could run a script to access your banking information directly from your browser session.
  
  1. You log into https://bank.com.
  2. You visit http://malicious.com.
  3. The malicious site runs a script that tries to access your session cookies or other sensitive data from https://bank.com.
  4. The script successfully reads your banking information and sends it to the attacker.
</details>

<details>
  <summary>Cross-Origin Resource Sharing (CORS)</summary>
  <br/>
  Cross-origin resource sharing (CORS) is an extension of the same-origin policy. You need CORS when you want to pull data from external APIs that are public or authorized. You also need CORS if you want to allow authorized third-party access to your own server resources.
  
</details>
<details>
  <summary>How does cross-origin resource sharing work?</summary>
  <br/>
  
  In browser terminology, the current browser URL is called the _current origin_ and the third-party URL is _cross-origin_.

  When you make a cross-origin request, for example:
  
  Consider a site called `https://news.example.com`. This _site_ wants to access resources from an API at `partner-api.com`. If developers at `https://partner-api.com` configure the cross-origin resource sharing (CORS) headers on their server by adding `new.example.com` to the allowed origins list. 

  _`Access-Control-Allow-Origin: https://news.example.com`_

  Once **CORS** access is configured, `news.example.com` can request resources from `partner-api.com`. For every request, `partner-api.com` will respond with `Access-Control-Allow-Credentials:"true"`. The browser then knows the communication is authorized and permits cross-origin access.

  When a web application attempts a cross-origin request, the browser evaluates the request based on the criteria for simple requests. If the request meets these criteria, it's sent directly without a preflight. Otherwise, the browser determines that a _**preflight**_ is necessary.
</details>

