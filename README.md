# Security Knownledge
## Oauth2

### Terminogoly

<details>
  <summary>Token Introspection</summary>
  <br/>

  Token introspection is a mechanism defined in OAuth 2.0 that allows resource servers to query an authorization server to validate and get information about an access token (or refresh token).

  _Example:_

  ```
  {
    "active": true,
    "scope": "read write profile",
    "client_id": "client123",
    "username": "johndoe",
    "token_type": "Bearer",
    "exp": 1728678400,
    "iat": 1728674800,
    "nbf": 1728674800,
    "sub": "user123",
    "aud": ["resource_server1", "resource_server2"],
    "iss": "https://auth.example.com",
    "jti": "abc123xyz",
    "custom_claims": {
      "roles": ["admin", "user"],
      "organization": "example_org"
    }
  }
  ```

</details>

### Authorization code

<details>
  <summary>How dose it work?</summary>
  <br/>
  
  ![](images/oauth_web_server_flow.png)
  
  + Ref: https://docs.oracle.com/cd/E50612_01/doc.11122/oauth_guide/content/oauth_flows.html
  + Ref: https://portswigger.net/web-security/oauth/grant-types
  
</details>
<details>
  <summary>Summary flow</summary>
  <br/>

  + **User Initiates Login:** The user clicks on a login button in your application.
  + **Redirect to Authorization Server:** The application redirects the user to the Authorization Server with a request that includes the `client ID`, `redirect URI`, `response type` (code), and `scope`.
  + **User Authenticates:** The user logs in and accept to the requested permissions.
  + **Authorization Code:** The Authorization Server redirects the user back to the application with an authorization code in the query string.
  + **Exchange Code for Tokens:** The application sends a request to the Authorization Server’s token endpoint, including the authorization code, `client ID`, `client secret`, and `redirect URI`.
  + **Access Token (and optionally Refresh Token):** The Authorization Server validates the authorization code and returns an access token (and optionally a refresh token).
  + **Access Protected Resources:** The application uses the access token to make API requests to access protected resources on behalf of the user.
</details>
<details>
  <summary>Refresh token flow</summary>
  <br/>

  + **Access Token Expiry:** When the access token expires, the client uses the refresh token to request a new access token.
  + **Token Endpoint Request:** The client sends a POST request to the token endpoint with the following parameters: `grant_type` (_`refresh_token`_), `refresh_token`, `client_id`, `client_secret`.
  + **New Tokens Issued:** The authorization server validates the refresh token and issues a **_new access token_**.
  + **Access Protected Resources:** The client uses the new access token to access protected resources.
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

### Fundamental concepts
<details>
  <summary>OpenId connect flow</summary>
  <br/>

  + **User Initiates Login:** The user clicks on a login button in your application.
  + **Redirect to Authorization Server:** The application redirects the user to the OpenID Provider (OP) with a request that includes the client ID, redirect URI, response type (code), and scope.
  + **User Authenticates:** The user logs in and consents to the requested permissions.
  + **Authorization Code:** The OP redirects the user back to the application with an authorization code in the query string.
  + **Exchange Code for Tokens:** The application sends a request to the OP’s token endpoint, including the authorization code, client ID, client secret, and redirect URI.
  + **ID Token and Access Token:** The OP validates the authorization code and returns an ID token and an access token.
  + **Verify ID Token:** The application verifies the ID token to ensure it is valid and issued by the OP.
  + **Access Protected Resources:** The application uses the access token to make API requests to access protected resources on behalf of the user.
  + **UserInfo Endpoint:** Optionally, the application can request additional user information from the UserInfo endpoint using the access token.
</details>
<details>
  <summary>Refresh token flow</summary>
  <br/>

  + **Access Token Expiry:** When the access token expires, the client uses the refresh token to request new tokens.
  + **Token Endpoint Request:** The client sends a POST request to the token endpoint with the following parameters: `grant_type` (_`refresh_token`_), `refresh_token`, `client_id`, `client_secret`.
  + **New Tokens Issued:** The authorization server validates the refresh token and issues a **_new access token_**, **_ID token_**.
  + **Access Protected Resources:** The client uses the new access token to access protected resources and the new ID token to verify the user’s identity.
</details>
<details>
  <summary>ID token structure</summary>
  <br/>
  
  An ID token is a JSON Web Token (JWT) that contains information about the authenticated user. It has three main parts: the header, payload, and signature.

  **Header:**
  + `alg`: The signing algorithm used, such as RS256.
  + `typ`: The type of token, which is JWT.

  **Payload:** The payload contains claims about the user and the authentication event. Some standard claims include:
  + `iss`: Issuer (the identity provider).
  + `sub`: Subject (the unique identifier for the user).
  + `aud`: Audience (the client ID of the application).
  + `exp`: Expiration time.
  + `iat`: Issued at time.
  + `name`: Full name of the user.
  + `email`: Email address of the user.
  + `picture`: URL of the user’s profile picture.

</details>
<details>
  <summary>UserInfo endpoint</summary>
  <br/>

  The UserInfo endpoint is a part of the OpenID Connect standard. It is used to retrieve claims about the authenticated user, such as their name, email, and other profile information. The UserInfo endpoint provides additional information about the user.

  **Location:**

  The URL for the UserInfo endpoint is provided by the OpenID Provider and can be found in the configuration document. Usually located at the `.well-known/openid-configuration` endpoint of the identity provider.

  **Usage**

  To call the UserInfo endpoint, we need an access token. This token is used to authorize the request to the UserInfo endpoint.

  **Content in ID token & UserInfo endpoint:**
  + The ID token contains claims about the authentication event and some basic user information.
  + The UserInfo endpoint can provide more detailed and extensive user profile information.
  
</details>

### OpenId Connect vs Oauth2

<details>
  <summary>OpenID Connect and OAuth2</summary>
  <br/>

  **OAuth2:**

  + OAuth 2.0 is focused on authorization (granting access to resources). 
  + OAuth 2.0 uses access tokens to grant access to resources.

  **OpenID Connect:**

  + OpenID Connect is focused on authentication (verifying user identity) and builds on OAuth 2.0 to provide this functionality.
  + OpenID Connect uses ID tokens to provide identity information about the user.

</details>
<details>
  <summary>Authorization Server and OpenID Provider</summary>
  <br/>

  + **Authorization Server:** This is a component responsible for authenticating the user, obtaining their consent, and issuing tokens (access tokens, refresh tokens, etc.).
  + **OpenID Provider (OP):** the OP is an extension of the _authorization server_. It not only handles the OAuth 2.0 tasks but also provides additional functionality for user authentication and identity management. 

</details>
<details>
  <summary>Access token vs ID Token</summary>
  <br/>

  The differences between an ID Token and an Access Token:

  _ID Token_
  + **Purpose:** Used to authenticate the user.
  + **Content:** Contains information about the user and the authentication event, such as the user’s identity, the issuer, and the expiration time.
  + **Format:** Typically a JSON Web Token (JWT).
  
  _Access Token_
  + **Purpose:** Used to authorize access to protected resources.
  + **Content:** Contains information needed to access resources, such as scopes and permissions.
  + **Format:** Can be a JWT or an string.
</details>

<details>
  <summary>Token Inspect Endpoint vs User Endpoint</summary>
  <br/>

  **Token inspect endpoint:**
  + Validates the token itself and returns information about it.
  + Retrieve details on token scope, expiration, and associated user or client permissions.
  + Information specific to the token, such as: `active`, `exp`, `scope`, `sub`.

  **User endpoint:**
  + Provides detailed information about the user associated with the authenticated session or token.
  + Information specific to the user, often including: `id`, `name`, `email`, `roles` or `permissions`, ... .
  
</details>

## JWT
### Fundamental concepts
<details>
  <summary>What is JWT</summary>
  <br/>

  JSON Web Token (JWT) is an open standard _(RFC 7519)_ that defines a way for securely transmitting information between parties as a JSON object. 

  **Structure of JWT:**

  + **Header:** This part typically consists of two elements
    + The type of the token, which is JWT.
    + The signing algorithm being used, such as HMAC SHA256 or RSA.
  ```
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

  + **Payload:** This part contains the claims. Claims are statements about an entity (typically, the user) and additional data. There are three types of claims:
    + **Registered claims:** Predefined claims like `iss` (issuer), `exp` (expiration time), `sub` (subject), and `aud` (audience).
    + **Public claims:** Custom claims that can be defined by users.
    + **Private claims:** Custom claims that are shared between parties that agree on using them.
  ```
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

  + **Signature:** This part is used to verify that the sender of the JWT to ensure the message wasn’t changed.
  ```
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret
  )
  ```

  ![](images/jwt_structure.png)
</details>
<details>
  <summary>JWT flow in Client-Server model</summary>
  <br/>
  
  ![](images/client-server-jwt.png)

  1. **User Authentication:** The user logs in with their credentials (e.g., username and password). Then the server verifies the credentials.
  2. **Token Creation:** The JWT is signed using a secret key or a public/private key pair.
  3. **Token Transmission:** The server sends the JWT back to the client (usually in the response body or as a cookie). Then the client stores the JWT, often in local storage or a cookie.
  4. **Subsequent Requests:** The client includes the JWT in the HTTP headers (usually in the Authorization header with the Bearer schema)
  5. **Token Verification:** The server verifies the JWT by checking its signature and ensuring it has not expired. If the token is valid, the server processes the request and sends the appropriate response.
  
  + Ref: https://cryptobook.nakov.com/digital-signatures
  
</details>
<details>
  <summary>Signing & Verification in JWT</summary>
  <br/>

  **Overview:**

  ![](images/how_signing_work.png)

  **Signing:**

  ![](images/jwt_signing.png)

  To create the signature (Signing), you need: _the encoded header_, _the encoded payload_, _a secret key_ (**private key**). Then the signature is created by using the specified algorithm and the secret key.
  ```
  HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    secret ()
  )
  ```
  
  **Verification:**
  
  The first step is to decode these parts. Then use the same algorithm specified in the header and the public key (_asymmetric keys_) to recreate the signature. Finally, compare the recreated signature with the one in the JWT. If they match, the token is valid.

  JWTs contain claims in the payload, such as the issuer (iss), expiration time (exp), and audience (aud). You should verify these claims to ensure:

  + **Expiration (exp):** The token is not expired.
  + **Issuer (iss):** The token was issued by a trusted authority.
  + **Audience (aud):** The token is intended for your application.

  
  + Ref: https://cryptobook.nakov.com/digital-signatures
  + Ref: https://stackoverflow.com/questions/18257185/how-does-a-public-key-verify-a-signature
</details>

### Detail about Signing & Verification
<details>
  <summary>Private & public key</summary>
  <br/>

  **Private key:** The private key is used to sign the JWT. This means that when a JWT is created, the issuer uses their private key to generate a digital signature based on the token’s header and payload.

  **Public Key:** The public key is used to verify the JWT’s signature. When a recipient receives a JWT, they use the issuer’s public key to verify that the token’s signature.
  
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
<details>
  <summary>Secret Key vs Private/Public Key Pair</summary>
  <br/>

  **Secret Key (Symmetric Key)**
  + The same secret key is used for both signing and verifying the JWT. This means that anyone with the secret key can create and verify tokens.
  + Often used in single-server applications or where the token verification happens within the same trusted environment.

  **Private/Public Key Pair (Asymmetric Keys)**
  + Ideal for distributed systems where different entities issue and verify tokens.
  + The private key is used to sign the JWT, and the public key is used to verify it. This ensures that only the issuer with the private key can create valid tokens, while anyone with the public key can verify them.
  
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

## Vault

### Fundamental concepts

<details>
  <summary>Features of Vault</summary>
  <br/>

  + **Data Encryption:** Vault provides encryption as a service, allowing you to encrypt and decrypt data without storing it.
  + **Secure Secret Storage:** Vault can store arbitrary key-value secrets, such as API keys, passwords, and certificates.
  + **High Availability (HA):** Vault supports high availability configurations to ensure that your Vault deployment remains operational event.
  
</details>
<details>
  <summary>HA</summary>
  <br/>

  High Availability (HA) in Vault ensures that your Vault deployment remains operational even if some of the servers fail.

  Vault operates with multiple server nodes:
  + **Active Node:** This node handles all the client requests and performs the operations.
  + **Standby Nodes:** These nodes are ready to take over if the active node fails.

   ![](images/HA_vault.png)

  + When Vault starts, it performs a leader election process to determine which node will be the active node. If the active node becomes unavailable, one of the standby nodes is automatically promoted to the active role.
  + Standby nodes can forward requests to the active node. This ensures that clients can still interact with Vault even if they connect to a standby node.
</details>

### Integrate with Spring boot
<details>
  <summary>Setup Vault with Spring boot</summary>
  <br/>

  Configure dependencies
  ```
  <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-vault-config</artifactId>
  </dependency>
  <dependency>
      <groupId>org.springframework.vault</groupId>
      <artifactId>spring-vault-core</artifactId>
  </dependency>
  ```

  Configure Vault in `application.yml`

  ```
  spring:
    cloud:
      vault:
        uri: http://127.0.0.1:8200
        token: s.YourVaultToken
        kv:
          enabled: true
          backend: secret
  ```

  + `enabled: true`: This tells Spring Cloud Vault to use the key-value secrets engine.
  + `backend: secret`: This specifies that the secrets are stored under the secret path in Vault. For example, if you store a secret at `secret/myapp`, you can access it in your Spring Boot application.

  **Create a Vault Configuration Class**

  ```
  @Configuration
  public class VaultConfig {
  
      @Bean
      public VaultTemplate vaultTemplate() {
          VaultEndpoint endpoint = VaultEndpoint.create("127.0.0.1", 8200);
          return new VaultTemplate(endpoint, new TokenAuthentication("s.YourVaultToken"));
      }
  }
  ```

  ```
  @RestController
  public class SecretController {
  
      @Value("${my.secret}")
      private String secret;
  
  }
  ```
  `@Value(“${my.secret}”)`: Injects the secret stored in Vault into the secret variable.
</details>
