# JWT - JSON Web Token

JWT stands for JSON Web Token. It is a compact, URL-safe means of representing claims between two parties. JWTs are commonly used to secure the transmission of information between parties in a web environment, typically for authentication and information exchange. The JWT specification is defined by RFC 7519[^1].

Here are the key components and concepts of JWT:

1. **JSON Format:** JWTs are represented as JSON objects that are easy to parse and generate. The JSON format makes them human-readable and easy to work with.
2. **Three Parts:** JWTs consist of three parts separated by dots (`.`): Header, Payload, and Signature.

    - **Header:** The header typically consists of two parts: the type of the token (JWT) and the signing algorithm being used, such as HMAC SHA256 or RSA.
    
    - **Payload:** The payload contains the claims. Claims are statements about an entity (typically, the user) and additional data. There are three types of claims: registered, public, and private claims.
    
    - **Signature:** To create the signature part, you take the encoded header, the encoded payload, a secret, the algorithm specified in the header, and sign that.

3. **Encoding:** Each of the three parts is Base64Url encoded, and the resulting strings are concatenated with periods between them. The final JWT looks like: `xxxxx.yyyyy.zzzzz`.
4. **Stateless and Self-contained:** JWTs are stateless, meaning that all the information needed is within the token itself. The server doesn't need to store the user's state. They are also self-contained, meaning that all the information needed is contained within the token.
5. **Use Cases:** JWTs are commonly used for authentication and information exchange between parties. For example, after a user logs in, a server could generate a JWT and send it to the client. The client can then include the JWT in the headers of subsequent requests to access protected resources. The server can verify the authenticity of the JWT using the stored secret key.
6. **Security Considerations:** While JWTs are widely used and versatile, it's important to handle them securely. For instance, the key used to sign the JWT should be kept secret, and HTTPS should be used to transmit JWTs to prevent man-in-the-middle attacks.

Here's a simple example of a JWT created on JWT Builder[^2]:

`eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9.eyJpc3MiOiJJbnNwZXIiLCJpYXQiOjE3MDMwMDgzMzgsImV4cCI6MjAxODU0MTEzOCwiYXVkIjoid3d3Lmluc3Blci5lZHUuYnIiLCJzdWIiOiJodW1iZXJ0b3JzQGluc3Blci5lZHUuYnIiLCJHaXZlbk5hbWUiOiJIdW1iZXJ0byIsIlN1cm5hbWUiOiJTYW5kbWFubiIsIkVtYWlsIjoiaHVtYmVydG9yc0BpbnNwZXIuZWR1LmJyIiwiUm9sZSI6IlByb2Zlc3NvciJ9.SsGdvR5GbYWTRbxY7IGxHt1vSxhkpRueBJWsi0lrPhJVCICp119QjU8F3QvHW0yF5tw-HhQ9RVh0l89t4M0LNw`

This JWT consists of three parts, decoded by [^3]:

- Header: `eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzUxMiJ9`

```json
{
  "typ": "JWT",
  "alg": "HS512"
}
```

- Payload: `eyJpc3MiOiJJbnNwZXIiLCJpYXQiOjE3MDMwMDgzMzgsImV4cCI6MjAxODU0MTEzOCwiYXVkIjoid3d3Lmluc3Blci5lZHUuYnIiLCJzdWIiOiJodW1iZXJ0b3JzQGluc3Blci5lZHUuYnIiLCJHaXZlbk5hbWUiOiJIdW1iZXJ0byIsIlN1cm5hbWUiOiJTYW5kbWFubiIsIkVtYWlsIjoiaHVtYmVydG9yc0BpbnNwZXIuZWR1LmJyIiwiUm9sZSI6IlByb2Zlc3NvciJ9`

```json
{
  "iss": "Insper",
  "iat": 1703008338,
  "exp": 2018541138,
  "aud": "www.insper.edu.br",
  "sub": "humbertors@insper.edu.br",
  "GivenName": "Humberto",
  "Surname": "Sandmann",
  "Email": "humbertors@insper.edu.br",
  "Role": "Professor"
}
```

- Signature: `SsGdvR5GbYWTRbxY7IGxHt1vSxhkpRueBJWsi0lrPhJVCICp119QjU8F3QvHW0yF5tw-HhQ9RVh0l89t4M0LNw`

```py
HMACSHA512(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  qwertyuiopasdfghjklzxcvbnm123456,
)
```

JWTs are widely used in web development due to their simplicity, flexibility, and support across various programming languages and frameworks. They are commonly used in token-based authentication systems.

[^1]: [RFC 7519 - JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519), 2015.
    
[^2]: [JWT - Builder](http://jwtbuilder.jamiekurtz.com).
    
[^3]: [jwt.io - JWT Verification](https://jwt.io/).

[^4]: [Unix Time Stamp - Epoch Converter](https://www.unixtimestamp.com)