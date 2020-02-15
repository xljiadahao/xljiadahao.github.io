---
title: Json Web Token (JWT)
tags:
  - json
  - security
  - identity
  - restful
  - distributed system
date: 2018-03-12
---

Json Web Token (JWT) plays a signifiant role as the secure identity, especially in the distributed architecure such as SOA (Service Oriented Architecture) systems. Here I would like to introduce JWT by the following steps. 1. Brief introduction of JWT, 2. Benefit of JWT, 3. Sample implementation to use JWT in Java.


a. `Brief introduction of JWT`
I would like to introduce JWT briefly. JWT is a secure information transmitting way based on the open standard (RFC 7519) by a json object. It contains 3 parts, Header, Payload, Signature seperated by dots (.). Therefore, a typical JWT token looks like the following format. `Header.Payload.Signature`. Let's break down to get the details of each part.
Header
```
{
  "alg": "HS256",
  "typ": "JWT"
}
```
The header generally consists of two parts: the type of the token, and the hashing algorithm being used. Then it will be getting base64 encoded to form the header part of the JWT.
Note: `Encoding is the public known way to tranfer the data from one format to another format.` Therefore, the header is getting base64 encoded, which means the header part of JWT can be considered as the `plain text`. Thus, we should NOT store the sensitive information in the JWT header.
Payload
```
{
  "sub": "third party authorization",
  "iss": "auth0",
  "exp": "1520239160663"
  "customclaims": 
  {
    "id": 33,
    "userName": "lei",
    "role": 
    {
      "roleName": "admin",
      "privileges": ["read", "write", "delete"]
    }
  }
}
```
Payload contains three types of claims: registered, public, and private claims. They are statements about an entity. Registered claims are a set of predefined claims with three characters claim name such as iss (issuer), exp (expiration time), sub (subject), aud (audience). Public claims can be defined at will by those using JWTs. Private claims are the custom claims for information sharing between parties like "customclaims" in the sample above. Then it will be getting base64 encoded to form the payload part of the JWT.
Note: Similar to JWT header, the payload is getting base64 encoded, which means the payload part of JWT can be considered as the `plain text`. Thus, we should NOT store the sensitive information in the JWT payload.
Signature
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
Use the algorithm specified in the header to sign the encoded header, the encoded payload with a secret. The signature is used to verify the message wasn't changed along the way. In the sample, we use HMAC SHA256 algorithm.
Note: `Signature typically takes use of hash function which is one way encryption.` It cannot be reverted back to get the secret by the signature. Any information in the token is getting changed, then it will result in generating different signature for the failed verification. Thus, the `secret` is very important for the verification, and it will be stored in the `server side` only securely. `During verification, the server will re-caculate the singature by first two parts of the JWT, the base64 encoded header and the base64 encoded payload, with the secret which is only known by the server, then compare the re-caculated signature with the sigature of the JWT`. 
Json Web Token (JWT)
```
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJhdXRoMCIsInN1YiI6InRoaXJkIHBhcnR5IGF1dGhvcml6YXRpb24iLCJjdXN0b21jbGFpbXMiOiJ7XCJpZFwiOjMzLFwidXNlck5hbWVcIjpcImxlaVwiLFwicm9sZVwiOntcInJvbGVOYW1lXCI6XCJhZG1pblwiLFwicHJpdmlsZWdlc1wiOltcInJlYWRcIixcIndyaXRlXCIsXCJkZWxldGVcIl19fSIsImV4cCI6MTUyMDcxNTcwODAzOH0.om5ZNtXDtsp6IUOErHcrE1kykiNEebjA55QCmxdWxnk
```
Therefore, the JWT like the sample above is the combination of the base64 encoded header, the base64 encoded payload, and the signiture signed with a secret.
Again, do note that with signed tokens, all the information contained within the token is exposed to users or other parties, even though they are unable to change it. This means you should not put secret information within the token.
As the general flow to use the JWT token, the user will login with the credential to get a JWT token, then the user can take use of the JWT token to access the protected resources. To find out more information about Json Web Token (JWT) or try out JWT token generation, decoding and verification online, browse the [jwt.io](https://jwt.io/) website.

b. `Benefit of JWT`
1 High efficiency. The server do not need to have the database access for authentication, and instead, it just need to re-caculate the signature in memory for verification.
2 Cross-Origin Resource Sharing. JWT token provides the stateless way for authentication, so it could not be an issue to access the resources cross multiple services as it doesn't use cookie and session, especially for the distributed architecture systems.
3 Light weight. JWT is a light weight format transmitting way between parties by the http request (mostly in the http header). It can exchange the necessary information within the JWT payload.

c `Sample implementation to use JWT in Java`
Here we use auth0 library for the implementation.
pom.xml
```
<dependency>
	<groupId>com.auth0</groupId>
	<artifactId>java-jwt</artifactId>
	<version>2.2.1</version>
</dependency>
```
Signing with a secret, return the JWT token as a String
```
public static<T> String sign(T object, long maxAge) throws Exception {
    final JWTSigner signer = new JWTSigner(SECRET);
    final Map<String, Object> claims = new HashMap<String, Object>();
    ObjectMapper mapper = new ObjectMapper();
    String jsonString = mapper.writeValueAsString(object);
    claims.put(CUSTOM_CLAIMS, jsonString);
    claims.put(EXPIRATION_TIME, System.currentTimeMillis() + maxAge);
    claims.put(ISSUER, "auth0");
    claims.put(SUBJECT, "third party authorization");
    return signer.sign(claims);
}
```
Unsigning with the same secret by signature verification
```
public static<T> T unsign(String jwt, Class<T> classT) throws Exception {
    final JWTVerifier verifier = new JWTVerifier(SECRET);
    final Map<String,Object> claims= verifier.verify(jwt);
    if (claims.containsKey(EXPIRATION_TIME) && claims.containsKey(ISSUER) 
            && claims.containsKey(SUBJECT) && claims.containsKey(CUSTOM_CLAIMS)) {
        long exp = (long) claims.get(EXPIRATION_TIME);
        System.out.println("EXPIRATION_TIME: " + sdf.format(new Date(exp)) + ", ISSUER: " 
                + (String) claims.get(ISSUER) + ", SUBJECT: " + (String) claims.get(SUBJECT));
        if (exp > System.currentTimeMillis()) {
            String privateClaims = (String) claims.get(CUSTOM_CLAIMS);
            ObjectMapper objectMapper = new ObjectMapper();
            return objectMapper.readValue(privateClaims, classT);
        }
    }
    return null;
}
```
Find more about the code snippets of JWT under the package _com.snippet.jwt_ of the Snippet repository in [GitHub](https://github.com/xljiadahao/SnippetInnovationWithSpringBoot.git).

The code snippets demo for the JWT token generation and verification.

1 login to get the JWT token within hateoas link
![request of token generation](/images/jwt/jwt_login_request.png "request for token generation")
![response of token generation](/images/jwt/jwt_login_response.png "response of token generation")
2 verify the token by getting the user information
![token verification](/images/jwt/jwt_verify_request_response.png "token verification")

#### Upcoming topic: Cryptography 101