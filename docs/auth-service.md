
As the platform has only one entrace point, it is

JWT is a decentralized 

The point of entrance of API is the gateway, then as suggested by [^5].

## Auth Service

- Responsabilities:
    - Registration:
    - Authentication:
    - Authorization:


Two Maven Projects

- Interfaces

- Implemmentation: resource

``` mermaid
classDiagram
  namespace Interface {
    class AuthController {
      <<interface>>
      register(RegisterIn)
      authenticate(CredentialIn)
      identify(String)
    }
    class RegisterIn {
      <<record>>
      String firstName
      String lastName
      String email
      String password
    }
    class CredentialIn {
      <<record>>
      String email
      String password
    }
  }
  namespace Resource {
    class AuthResource {
      <<REST API>>
      -authService
    }
    class AuthService {
      <<service>>
      -registerRepository
      -userRepository
      register(Register)
      authenticate(Credential)
      identify(Session)
    }
    class RegisterRepository {
      <<interface>>
    }
    class RegisterEntity {
      <<entity>>
    }
    class UserRepository {
      <<interface>>
    }
    class UserEntity {
      <<entity>>
    }
  }
  AuthController <|-- AuthResource
  AuthResource o-- AuthService
  AuthService o-- RegisterRepository
  AuthService o-- UserRepository
  RegisterRepository "1" --> "0..*" RegisterEntity
  UserRepository "1" --> "0..*" UserEntity
```


## Addtional Material

- [JSON Web Token](./jwt.md)

- <a href="https://www.youtube.com/watch?v=5w-YCcOjPD0" target="_blank">Fernanda Kipper - Autenticação e Autorização com Spring Security e JWT Tokens</a></i>

    [![](https://img.youtube.com/vi/5w-YCcOjPD0/0.jpg){ width=80% }](https://www.youtube.com/watch?v=5w-YCcOjPD0){:target="_blank"}


[^5]: DELANTHA, R., [Spring Cloud Gateway security with JWT](https://medium.com/@rajithgama/spring-cloud-gateway-security-with-jwt-23045ba59b8a){:target="_blank"}, 2023.
