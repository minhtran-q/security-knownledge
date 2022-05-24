# Security Knownledge
## Oauth2

### Authorization code

<details>
  <summary>How dose it work?</summary>
  <br/>
  
  + Ref: https://docs.oracle.com/cd/E50612_01/doc.11122/oauth_guide/content/oauth_flows.html
  
</details>
<details>
  <summary>What is the purpose of authorization code in OAuth</summary>
  <br/>
  
  It's possible to do it with a single request - it's called the _**implicit flow**_ then.
  
  The general idea of using access code (authorization flow) instead of directly returning the tokens is to **hide** them from the end user. The second request is done usually by the backend server instead of a browser.
  
  This exchange of _authorization code_ doesn't involve the user’s browser so there is no way access tokens are stored in history of the browser.

  + Ref: https://stackoverflow.com/questions/53995441/what-is-the-purpose-of-authorization-code-in-oauth
  + Ref: https://stackoverflow.com/questions/7522831/what-is-the-purpose-of-the-implicit-grant-authorization-type-in-oauth-2
  + Ref: https://www.quora.com/Why-does-OAuth-server-return-a-authorization-code-instead-of-access-token-in-the-first-step
  
</details>

### Client credential
### Resource owner password credential
### Implicit

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
