# Script: analyze-gateway-contracts

**Modo:** Modo 2 — Contratos de Gateway (Pipeline)
**Entrada:** Arquivos de especificação OpenAPI em `/contratos/*.yaml`
**Saída:** Lista estruturada de APIs detectadas com metadados

---

## Objetivo

Analisar os arquivos YAML OpenAPI/Swagger em `/contratos` para identificar todos os endpoints HTTP expostos, seus métodos, caminhos, schemas, requisitos de autenticação e dependências entre endpoints.

---

## Procedimento

### Passo 1 — Listar os Arquivos de Contrato

Enumerar todos os arquivos `.yaml` e `.yml` em `/contratos`:

```
/contratos/api1.yaml
/contratos/api2.yaml
/contratos/api3.yaml
```

Processar cada arquivo sequencialmente. Se houver múltiplos arquivos, mesclar as listas de endpoints e resolver dependências entre arquivos.

### Passo 2 — Analisar a Estrutura OpenAPI

Para cada arquivo de contrato, extrair os campos raiz do OpenAPI:

```yaml
openapi: "3.0.x"        # ou swagger: "2.0"
info:
  title: <título da API>
  version: <versão>
servers:
  - url: <base_url>
paths:
  /endpoint:
    get: ...
    post: ...
components:
  schemas: ...
  securitySchemes: ...
```

### Passo 3 — Extrair Metadados dos Endpoints

Para cada combinação de path e método HTTP, extrair:

```
- name: derivado do operationId ou path + método (ex.: "get-products")
- method: GET, POST, PUT, DELETE, PATCH
- path: o caminho da URL (ex.: /products/{id})
- path_params: parâmetros com `in: path`
- query_params: parâmetros com `in: query`
- request_body_fields: campos do schema do requestBody (POST/PUT/PATCH)
- response_fields: campos no schema de resposta 200/201 (especialmente IDs)
- requires_auth: true se o endpoint possui esquema de segurança aplicado
- depends_on: inferido a partir de parâmetros de path e referências de campos do corpo
- source_file: qual arquivo de contrato definiu este endpoint
```

#### Regras de derivação de nome:
1. Usar `operationId` se presente, convertido para kebab-case
2. Caso contrário, derivar do método + path: `GET /products/{id}` → `get-product-by-id`

### Passo 4 — Detectar Esquemas de Autenticação

Inspecionar `components.securitySchemes`:

| Tipo de Esquema | Tratamento |
|---|---|
| `http` com `bearer` | Adicionar cabeçalho `Authorization: Bearer ${auth_token}` |
| `apiKey` no cabeçalho | Adicionar o cabeçalho correspondente com `${auth_token}` |
| `oauth2` | Tratar como Bearer token; assumir token obtido externamente |
| `http` com `basic` | Adicionar cabeçalho `Authorization: Basic ${auth_token}` |

Se algum endpoint exigir autenticação, uma etapa de autenticação deve ser a primeira na sequência de execução.

### Passo 5 — Identificar Dependências

Um endpoint **depende de** outro quando:

1. **Dependência de parâmetro de path:** O path contém `{paramName}` e o schema de resposta de outro endpoint inclui um campo chamado `id`, `paramName` ou equivalente lógico.
   - Exemplo: `GET /orders/{orderId}` depende de `POST /orders` que retorna `orderId`

2. **Dependência de corpo de requisição:** O corpo da requisição referencia um ID ou chave que corresponde a um campo na resposta de outro endpoint.
   - Exemplo: `POST /cart/items` com corpo `{ "productId": "..." }` depende de `GET /products` que retorna `id`

3. **Dependência de fluxo de negócio:** Baseada em lógica de negócio (ex.: não é possível fazer checkout sem itens no carrinho).

### Passo 6 — Ordenação Topológica

Realizar ordenação topológica no grafo de dependências para determinar a ordem de execução dos samplers no JMeter.

### Passo 7 — Gerar Resumo de Saída

Produzir o mesmo formato de saída estruturada que `analyze-app-apis.md`:

```yaml
source: gateway-contracts
contracts:
  - /contratos/auth.yaml
  - /contratos/products.yaml
  - /contratos/orders.yaml

endpoints:
  - name: authenticate
    method: POST
    path: /auth/token
    requires_auth: false
    response_fields:
      - access_token
    depends_on: []

  - name: list-products
    method: GET
    path: /products
    requires_auth: true
    response_fields:
      - id
      - sku
    depends_on:
      - authenticate

  - name: create-order
    method: POST
    path: /orders
    requires_auth: true
    request_body_fields:
      - product_id
      - quantity
    depends_on:
      - list-products

execution_order:
  - authenticate
  - list-products
  - create-order
```

---

## Observações

- Ignorar endpoints marcados como `deprecated: true` na especificação OpenAPI.
- Ignorar endpoints utilitários: health checks, readiness probes, metrics.
- Se `servers[0].url` estiver presente, usá-lo como valor padrão do placeholder `base_url` no arquivo `.properties` gerado.
- Se schemas usam `$ref`, resolver a referência a partir de `components/schemas` no mesmo arquivo.
- Resolução de `$ref` entre arquivos: se um `$ref` aponta para outro arquivo de contrato, carregá-lo e resolvê-lo.
- Se `operationId` estiver ausente em muitos endpoints, usar uma convenção de nomenclatura consistente: `{method}-{path-segments-kebab-case}`.
