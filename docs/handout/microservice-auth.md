
## Endpoints

A fim do sistema possuir um controle de acesso, é conveniente a criação de um microsserviço Auth, que será responsável pelo cadastro de usuários do sistema. Tal microsserviço pode ter, ao menos, os seguintes endpoints:

### Registro
---
```
POST /auth/register
```
Request
``` json
{
    "name": "Antonio do Estudo",
    "email": "acme@insper.edu.br",
    "password": "123@"
}
```
Response

| code | body | 
|--|--|
| 201 | `{ "id": "45d16201-12a4-48bf-8c84-df768fdc4878" }`{:.json} |

#### Sequence Diagram

```
POST /auth/register
```

``` mermaid
sequenceDiagram
autonumber
actor User
User->>+Auth: register(RegisterIn)
Auth->>+Account: create(AccountIn)
Account->>-Auth: returns the new account (AccountOut)
Auth->>-User: returns 201
```

### Autenticação
---
``` 
POST /auth/login
```
Request
``` json
{
    "email": "acme@insper.edu.br",
    "password": "123@"
}
```
Response

| code | body | 
|--|--|
| 201 | `{ "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiI0NWQxNjIwMS0xMmE0LTQ4YmYtOGM4NC1kZjc2OGZkYzQ4NzgiLCJuYW1lIjoiQW50b25pbyBkbyBFc3R1ZG8iLCJpYXQiOjE1MTYyMzkwMjIsInJvbGUiOiJyZWd1bGFyIn0.8eiTZjXGUFrseBP5J91UdDctw-Flp7HP-PAp1eO8f1M" }`{:.json} |
| 403 | | 

#### Sequence Diagram

```
POST /auth/login
```

``` mermaid
sequenceDiagram
autonumber
actor User
User->>+Auth: authenticate(CredentiaIn)
Auth->>+Account: login(LoginIn)
critical validated
    Account->>-Auth: returns the account
option denied
    Auth-->>User: unauthorized message
end  
Auth->>Auth: generates a token
Auth->>-User: returns the token
User->>User: stores the token to use for the next requests
```

## Class Diagram

Exemplo para o microsserviço **Auth**.

``` mermaid
classDiagram
  namespace Interface {
    class AuthController {
      <<interface>>
      register(RegisterIn)
      authenticate(CredentialIn): LoginOut
    }
    class RegisterIn {
      <<record>>
      String name
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
      register(RegisterIn)
      authenticate(CredentialIn)
    }
  }
  AuthController <|-- AuthResource
  AuthResource o-- AuthService
```

### Exemplo de uma implementação da interface **AuthController**

``` java
package store.auth;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;

@FeignClient("store-auth")
public interface AuthController {

    @PostMapping("/auth/register")
    ResponseEntity<?> create (
        @RequestBody(required = true) RegisterIn in
    );

    @PostMapping("/auth/login")
    ResponseEntity<LoginOut> authenticate (
        @RequestBody(required = true) Credential in
    );
}
```

## POM dependecy

Note que esse microsserviço possui dependência de outro, o [Account](./microservice-account.md), além da dependência da interface do próprio microsserviço. Logo, se torna necessário explicitar essa dependência no `pom.xml` do microsserviço [Auth](./microservice-auth.md).

``` xml
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>auth</artifactId>
  <version>${project.version}</version>
</dependency>
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>account</artifactId>
  <version>${project.version}</version>
</dependency>
```

## **_NICE TO HAVE_**

> o projeto da disciplina pode ter um microsserviço de registro que valide email ou SMS para criar a conta.