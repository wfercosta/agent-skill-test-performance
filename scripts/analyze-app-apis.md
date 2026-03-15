# Script: analyze-app-apis

**Modo:** Modo 1 — Codebase (Pipeline)
**Entrada:** Diretório `/app`
**Saída:** Lista estruturada de APIs detectadas com metadados

---

## Objetivo

Varrer o código-fonte da aplicação em `/app` para identificar todos os endpoints HTTP expostos, seus métodos HTTP, caminhos, requisitos de autenticação, formatos de dados de requisição/resposta e dependências entre endpoints.

---

## Procedimento

### Passo 1 — Detectar Framework e Linguagem

Inspecionar os arquivos em `/app` para identificar a linguagem de programação e o framework web:

| Sinal | Framework |
|---|---|
| `pom.xml` ou `build.gradle` com `spring-boot` | Java / Spring Boot |
| `package.json` com `express`, `fastify`, `koa`, `hapi` | Node.js |
| `requirements.txt` ou `pyproject.toml` com `fastapi`, `flask`, `django` | Python |
| `go.mod` com `gin`, `echo`, `chi`, `fiber` | Go |
| `Gemfile` com `rails` ou `sinatra` | Ruby |

### Passo 2 — Localizar Definições de Rotas/Controllers

Buscar arquivos de rota e controller com base no framework detectado:

| Framework | Padrão de Busca |
|---|---|
| Spring Boot | `@RestController`, `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@RequestMapping` |
| Express.js | `router.get`, `router.post`, `app.get`, `app.post`, `app.use` |
| FastAPI | `@app.get`, `@app.post`, `@router.get`, `@router.post` |
| Gin (Go) | `r.GET`, `r.POST`, `r.PUT`, `r.DELETE`, `r.Group` |
| Rails | `routes.rb`, `resources`, `get`, `post`, `namespace` |

### Passo 3 — Extrair Metadados dos Endpoints

Para cada endpoint detectado, extrair:

```
- name: nome legível (ex.: "get-products")
- method: método HTTP (GET, POST, PUT, DELETE, PATCH)
- path: caminho da URL (ex.: /api/v1/products)
- path_params: lista de parâmetros de path (ex.: {id})
- query_params: lista de parâmetros de query
- request_body_fields: campos relevantes do corpo da requisição (se POST/PUT)
- response_fields: campos relevantes retornados (especialmente IDs para encadeamento)
- requires_auth: booleano — este endpoint exige token/sessão?
- depends_on: lista de nomes de endpoints cujos dados de resposta este endpoint necessita
```

### Passo 4 — Identificar Dependências

Um endpoint **depende de** outro quando:
- Usa um parâmetro de path (ex.: `/orders/{orderId}`) cujo valor deve vir de uma resposta anterior
- Exige sessão/token obtido de um endpoint de login
- Seu corpo de requisição inclui IDs ou chaves retornados por um endpoint anterior

Construir um mapa de dependências:
```
authenticate → get-products
authenticate → get-user
get-products → add-to-cart
add-to-cart → checkout
```

### Passo 5 — Determinar a Ordem de Execução

Realizar uma ordenação topológica do grafo de dependências para estabelecer a ordem de execução dos samplers no script JMeter.

Se não houver dependências, preservar a ordem de descoberta.

### Passo 6 — Gerar Resumo de Saída

Produzir um resumo estruturado:

```yaml
framework: <framework detectado>
language: <linguagem detectada>

endpoints:
  - name: authenticate
    method: POST
    path: /api/v1/auth/token
    requires_auth: false
    response_fields:
      - access_token
    depends_on: []

  - name: get-products
    method: GET
    path: /api/v1/products
    requires_auth: true
    response_fields:
      - id
      - name
    depends_on:
      - authenticate

  - name: checkout
    method: POST
    path: /api/v1/orders
    requires_auth: true
    request_body_fields:
      - product_id
      - quantity
    depends_on:
      - get-products

execution_order:
  - authenticate
  - get-products
  - checkout
```

---

## Observações

- Se endpoints de autenticação (login, token, OAuth) forem detectados, devem sempre ser os primeiros na ordem de execução.
- Se um endpoint usa JWT token ou API key, marcá-lo como `requires_auth: true`.
- Parâmetros de path como `{id}`, `{userId}`, `{orderId}` indicam dependências de dados — rastrear de onde esses valores se originam.
- Ignorar endpoints internos/privados (health checks, actuator, metrics), a menos que sejam especificamente relevantes para o cenário.
- Ignorar endpoints com prefixo `/internal`, `/actuator`, `/health`, `/metrics`, `/swagger`, `/openapi`.
