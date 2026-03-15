# Script: generate-test-data

**Entrada:** Objeto de cenário de `generate-test-scenario.md`
**Saída:** `<scenario.file_prefix>.csv` — arquivo de massa de dados para o teste

---

## Objetivo

Gerar um arquivo CSV com dados de teste realistas que serão consumidos pelo CSV Data Set Config do JMeter durante a execução do teste. O CSV fornece entradas parametrizadas para evitar o envio de dados idênticos em cada requisição.

---

## Procedimento

### Passo 1 — Determinar os Cabeçalhos CSV

Usar `scenario.csv_headers` do objeto de cenário.

Sempre incluir no mínimo:
- `user_id`
- `account_id`

Adicionar colunas com base nos endpoints detectados:
- `product_id` — se algum endpoint envolver produtos
- `order_id` — se algum endpoint envolver pedidos (pedidos pré-existentes)
- `cart_id` — se algum endpoint envolver carrinhos
- Quaisquer outros campos de ID exigidos por parâmetros de path ou corpos de requisição

**NÃO incluir** campos que são extraídos em tempo de execução das respostas da API (ex.: `access_token`, `orderId` criado dinamicamente).

### Passo 2 — Gerar as Linhas de Dados

Gerar pelo menos **10 linhas** de dados de exemplo realistas.

Regras de geração de dados por tipo de coluna:

#### `user_id`
- Formato: UUID v4 ou inteiro sequencial
- Exemplos UUID: `a1b2c3d4-e5f6-7890-abcd-ef1234567890`
- Exemplos inteiros: `1001`, `1002`, ..., `1010`

#### `account_id`
- Formato: string alfanumérica ou UUID
- Exemplo: `ACC-001`, `ACC-002`, ..., ou UUIDs

#### `product_id`
- Formato: código de produto alfanumérico ou UUID
- Exemplo: `PROD-001`, `PROD-002`, ..., ou UUIDs

#### `order_id`
- Formato: alfanumérico ou UUID
- Exemplo: `ORD-001`, `ORD-002`, ...

#### `quantity`
- Inteiro entre 1 e 10

#### `email`
- Formato: `user{n}@testdomain.local`
- Exemplo: `user1@testdomain.local`

#### Genérico `{entidade}_id`
- Seguir o mesmo padrão acima para aquela entidade

### Passo 3 — Formatar a Saída

Regras de formato CSV:
- Primeira linha: cabeçalho (nomes das colunas separados por vírgula)
- Linhas seguintes: valores dos dados (separados por vírgula)
- Sem espaços após vírgulas
- Sem aspas a menos que o valor contenha uma vírgula
- Codificação UTF-8

### Passo 4 — Escrever os Arquivos

Escrever o CSV em:
- `tests/performance/dev/<scenario.file_prefix>.csv`
- `tests/performance/hom/<scenario.file_prefix>.csv`

Ambos os arquivos contêm os mesmos dados (as diferenças de ambiente estão nos arquivos `.properties`).

---

## Exemplo de Saída

Para um cenário envolvendo autenticação, navegação de produtos e pedidos:

```csv
user_id,account_id,product_id,quantity
a1b2c3d4-e5f6-7890-abcd-ef1234567890,ACC-001,PROD-001,2
b2c3d4e5-f6a7-8901-bcde-f12345678901,ACC-002,PROD-002,1
c3d4e5f6-a7b8-9012-cdef-123456789012,ACC-003,PROD-003,3
d4e5f6a7-b8c9-0123-defa-234567890123,ACC-004,PROD-001,1
e5f6a7b8-c9d0-1234-efab-345678901234,ACC-005,PROD-004,5
f6a7b8c9-d0e1-2345-fabc-456789012345,ACC-006,PROD-002,2
a7b8c9d0-e1f2-3456-abcd-567890123456,ACC-007,PROD-005,1
b8c9d0e1-f2a3-4567-bcde-678901234567,ACC-008,PROD-003,4
c9d0e1f2-a3b4-5678-cdef-789012345678,ACC-009,PROD-001,2
d0e1f2a3-b4c5-6789-defa-890123456789,ACC-010,PROD-006,3
```

---

## Observações

- Os dados devem ser suficientemente variados para que o JMeter percorra linhas diferentes a cada iteração
- Usar dados fictícios com aparência realista — sem PII real, UUIDs reais ou números de conta reais
- Os nomes das colunas no CSV devem corresponder exatamente ao `variableNames` configurado no CSV Data Set Config do JMeter
- Se o cenário envolver login com usuário/senha em vez de token, adicionar colunas `username` e `password` com valores placeholder
