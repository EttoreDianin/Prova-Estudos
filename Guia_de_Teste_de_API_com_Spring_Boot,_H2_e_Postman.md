# Guia de Teste de API com Spring Boot, H2 e Postman

Este guia detalha o processo de configuração de uma API RESTful em Java usando **Spring Boot** e o banco de dados em memória **H2** para testes rápidos, além de como utilizar o **Postman** para realizar requisições `POST` e validar a funcionalidade.

O uso do H2 em modo em memória é ideal para testes, pois ele é iniciado e encerrado com a aplicação, garantindo um estado limpo a cada execução.

## 1. Configuração do Projeto Spring Boot

O Spring Boot simplifica a criação de aplicações Java. Usaremos o [Spring Initializr](https://start.spring.io/) para gerar a estrutura básica do projeto.

### 1.1. Dependências Necessárias

Selecione as seguintes dependências no Initializr (ou adicione ao seu `pom.xml` se estiver usando Maven):

| Dependência | Descrição |
| :--- | :--- |
| **Spring Web** | Para construir APIs RESTful (Controladores). |
| **Spring Data JPA** | Para persistência de dados e interação com o banco de dados. |
| **H2 Database** | O banco de dados em memória que usaremos para testes. |
| **Lombok** | (Opcional, mas recomendado) Reduz a verbosidade do código Java (getters, setters, construtores). |

### 1.2. Configuração do H2 (`src/main/resources/application.properties`)

Para garantir que o H2 funcione como um banco de dados em memória e que o console de gerenciamento esteja acessível, adicione as seguintes linhas ao seu arquivo `application.properties`:

```properties
# Configuração do Servidor
server.port=8080

# Configuração do H2 Database
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password

# Configuração do JPA/Hibernate
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.jpa.hibernate.ddl-auto=update
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

Com esta configuração, o banco de dados será criado automaticamente na memória com o nome `testdb` ao iniciar a aplicação. Você pode acessá-lo em `http://localhost:8080/h2-console` usando a URL JDBC `jdbc:h2:mem:testdb`.

### 1.3. Estrutura da API de Exemplo

Vamos criar uma API simples para gerenciar um objeto `Produto`.

#### A. Entidade (`Produto.java`)

Esta classe representa a tabela no banco de dados H2.

```java
package com.exemplo.apitest.model;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import lombok.Data;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Entity
@Data // Gera Getters, Setters, toString, equals e hashCode (Lombok)
@NoArgsConstructor
@AllArgsConstructor
public class Produto {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String nome;
    private Double preco;
}
```

#### B. Repositório (`ProdutoRepository.java`)

A interface que permite a interação com o banco de dados (CRUD). O Spring Data JPA implementa os métodos automaticamente.

```java
package com.exemplo.apitest.repository;

import com.exemplo.apitest.model.Produto;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface ProdutoRepository extends JpaRepository<Produto, Long> {
    // Métodos CRUD básicos já estão disponíveis (save, findById, findAll, etc.)
}
```

#### C. Controlador (`ProdutoController.java`)

O controlador REST que expõe o endpoint `POST` para criar um novo produto.

```java
package com.exemplo.apitest.controller;

import com.exemplo.apitest.model.Produto;
import com.exemplo.apitest.repository.ProdutoRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/produtos")
public class ProdutoController {

    @Autowired
    private ProdutoRepository produtoRepository;

    /**
     * Endpoint para criar um novo Produto (Teste POST)
     * @param produto O objeto Produto enviado no corpo da requisição.
     * @return O Produto salvo com o ID gerado e o status HTTP 201 (Created).
     */
    @PostMapping
    public ResponseEntity<Produto> criarProduto(@RequestBody Produto produto) {
        Produto novoProduto = produtoRepository.save(produto);
        return new ResponseEntity<>(novoProduto, HttpStatus.CREATED);
    }

    /**
     * Endpoint para listar todos os Produtos (Teste GET - Opcional)
     */
    @GetMapping
    public ResponseEntity<Iterable<Produto>> listarProdutos() {
        return new ResponseEntity<>(produtoRepository.findAll(), HttpStatus.OK);
    }
}
```

Com o servidor Spring Boot iniciado, a API estará pronta para receber requisições. O banco de dados H2 será preenchido com os dados que você enviar via `POST`.

[1]: https://spring.io/guides/tutorials/rest "Building REST services with Spring"
[2]: https://www.baeldung.com/spring-boot-h2-database "Spring Boot With H2 Database"
[3]: https://www.postman.com/api-evangelist/general-services-administration-gsa/documentation/mj1m8wb/spring-boot-rest-api "Spring Boot REST API | Documentation"

## 2. Testando o Endpoint POST com Postman

O Postman é uma ferramenta essencial para testar APIs. Ele permite enviar requisições HTTP e inspecionar as respostas de forma fácil e intuitiva.

### 2.1. Preparação da Requisição POST

Com a aplicação Spring Boot rodando (o servidor deve estar ativo, geralmente na porta `8080`), siga os passos no Postman:

1.  **Crie uma Nova Requisição:** Clique em `New` e selecione `HTTP Request`.
2.  **Selecione o Método:** Mude o método HTTP de `GET` para **`POST`**.
3.  **Defina a URL:** Insira a URL completa do endpoint que criamos: `http://localhost:8080/api/produtos`.
4.  **Configure o Corpo (Body):**
    *   Clique na aba **`Body`**.
    *   Selecione a opção **`raw`**.
    *   No menu suspenso ao lado de `raw`, selecione **`JSON`** (para indicar que o corpo da requisição é um objeto JSON).
    *   Insira o JSON que representa o objeto `Produto` que você deseja criar.

### 2.2. Exemplo de Requisição

O corpo da requisição deve seguir a estrutura da sua entidade `Produto` (ignorando o campo `id`, que é gerado automaticamente):

```json
{
    "nome": "Smartphone Modelo X",
    "preco": 1299.99
}
```

### 2.3. Execução e Verificação

1.  **Envie a Requisição:** Clique no botão **`Send`**.
2.  **Verifique o Status:** A resposta deve retornar o código de status **`201 Created`**. Este código indica que a requisição foi bem-sucedida e um novo recurso foi criado no servidor.
3.  **Inspecione o Corpo da Resposta:** O corpo da resposta (retornado pelo método `criarProduto` do controlador) deve conter o objeto `Produto` que você enviou, mas agora com o campo `id` preenchido, confirmando que ele foi salvo no banco de dados H2.

```json
{
    "id": 1,
    "nome": "Smartphone Modelo X",
    "preco": 1299.99
}
```

### 2.4. Validação no Banco de Dados H2 (Opcional)

Para confirmar que o dado foi persistido no H2:

1.  Acesse o console do H2 no seu navegador: `http://localhost:8080/h2-console`.
2.  Use as credenciais configuradas (`jdbc:h2:mem:testdb`, Usuário: `sa`, Senha: `password`) e clique em **`Connect`**.
3.  Execute a seguinte consulta SQL: `SELECT * FROM PRODUTO;`
4.  Você deverá ver a linha correspondente ao produto que você acabou de criar via Postman, confirmando o sucesso do teste de integração entre a API e o banco de dados H2.

## Referências

[1]: https://spring.io/guides/tutorials/rest "Building REST services with Spring"
[2]: https://www.baeldung.com/spring-boot-h2-database "Spring Boot With H2 Database"
[3]: https://www.postman.com/api-evangelist/general-services-administration-gsa/documentation/mj1m8wb/spring-boot-rest-api "Spring Boot REST API | Documentation"
