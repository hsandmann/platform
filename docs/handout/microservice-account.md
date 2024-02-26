
## Endpoints

Microsserviço para gerenciamento de contas.

### CREATE
---
```
POST /accounts
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
| 201 | `{ "id": "45d16201-12a4-48bf-8c84-df768fdc4878", "name": "Antonio do Estudo", "email": "acme@insper.edu.br" }`{:.json} |
| 401 | | 

### READ
---
```
GET /accounts
```
Response

| code | body | 
|--|--|
| 200 | `{ "id": "45d16201-12a4-48bf-8c84-df768fdc4878", "name": "Antonio do Estudo", "email": "acme@insper.edu.br" }`{:.json} |
| 401 | | 

## Class Diagram

Exemplo para o microsserviço **Account**.

``` mermaid
classDiagram
  namespace Interface {
    class AccountController {
      <<interface>>
      create(AccountIn)
      read(String id): AccountOut
      update(String id, AccountIn)
      delete(String id)
      findByEmailAndPassword(AccountIn)
    }
    class AccountIn {
      <<record>>
      String name
      String email
      String password
    }
    class AccountOut {
      <<record>>
      String id
      String name
      String email
    }
  }
  namespace Resource {
    class AccountResource {
      <<REST API>>
      -accountService
    }
    class AccountService {
      <<service>>
      -accountRepository
      create(Account)
    }
    class AccountRepository {
      <<nterface>>
      findByEmailAndHash(String, String)
    }
    class AccountModel {
      <<entity>>
      String id
      String name
      String email
      String hash
    }
    class Account {
      <<dto>>
      String id
      String name
      String email
      String password
    }
  }
  AccountController <|-- AccountResource
  AccountResource o-- AccountService
  AccountService o-- AccountRepository
```

## POM dependecy

Note que esse microsserviço possui dependência da interface, o **Account**. Logo, se torna necessário explicitar essa dependência no `pom.xml` do microsserviço [Account](./microservice-account.md).

``` xml
<dependency>
  <groupId>insper.store</groupId>
  <artifactId>account</artifactId>
  <version>${project.version}</version>
</dependency>
```
