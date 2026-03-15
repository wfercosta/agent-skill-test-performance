# Script: generate-jmeter-script

**Entrada:** Objeto de cenário de `generate-test-scenario.md`
**Saída:** `<scenario.file_prefix>.jmx` — arquivo XML de plano de teste JMeter válido

---

## Objetivo

Gerar um plano de teste Apache JMeter 5.x completo e válido (`.jmx`) com base nos parâmetros do cenário e na lista de endpoints produzidos pelos scripts de análise e de cenário.

---

## Procedimento

### Passo 1 — Inicializar o Plano de Teste

Usar a estrutura XML base de `references/jmeter-template.md`.

Definir `testname` como `scenario.name`.

### Passo 2 — Configurar o CSV Data Set

- `filename`: `${scenario.file_prefix}.csv`
- `variableNames`: lista separada por vírgula de `scenario.csv_headers`
- `ignoreFirstLine`: `true`
- `recycle`: `true`
- `shareMode`: `shareMode.all`

### Passo 3 — Configurar os Padrões de Requisição HTTP

- `domain`: `${base_url}` (carregado do `.properties` em tempo de execução)
- `protocol`: `https`
- `connect_timeout`: `5000`
- `response_timeout`: `30000`

### Passo 4 — Configurar o Gerenciador de Cabeçalhos HTTP

Sempre incluir:
```xml
Content-Type: application/json
Authorization: Bearer ${auth_token}
```

Adicionar o cabeçalho `client_id` se detectado na análise:
```xml
X-Client-Id: ${client_id}
```

### Passo 5 — Configurar o Thread Group

Mapear a partir dos parâmetros de execução do cenário:

| Propriedade JMeter | Origem |
|---|---|
| `num_threads` | `scenario.execution.concurrency` |
| `ramp_time` | `scenario.execution.ramp_up_seconds` |
| `duration` | `scenario.execution.hold_seconds` |
| `scheduler` | `true` |
| `on_sample_error` | `continue` |

### Passo 6 — Gerar os Samplers HTTP

Para cada endpoint em `scenario.execution_order`:

#### 6.1 — Determinar o Tipo de Sampler

- Se `method` for GET ou DELETE → usar o padrão GET de `references/jmeter-template.md`
- Se `method` for POST, PUT ou PATCH → usar o padrão POST com corpo JSON

#### 6.2 — Construir o Path

Substituir parâmetros de path por referências a variáveis JMeter:
```
/api/v1/products/{productId}  →  /api/v1/products/${productId}
/api/v1/users/{userId}/orders →  /api/v1/users/${userId}/orders
```

#### 6.3 — Construir o Corpo da Requisição

Para endpoints POST/PUT/PATCH, construir um corpo JSON a partir de `request_body_fields`:

```json
{
  "userId": "${user_id}",
  "productId": "${product_id}",
  "quantity": "${quantity}"
}
```

Usar a sintaxe de variáveis JMeter `${variable_name}` para todos os valores dinâmicos.

#### 6.4 — Adicionar Response Assertion

- Para GET → verificar código de resposta `200`
- Para POST → verificar código de resposta `200` ou `201`
- Para PUT/PATCH → verificar código de resposta `200` ou `204`
- Para DELETE → verificar código de resposta `200` ou `204`

#### 6.5 — Adicionar JSON Extractor (se o endpoint é uma dependência)

Se os campos de resposta deste endpoint são necessários por um endpoint downstream:

```xml
<JSONPathExtractor>
  referenceNames: access_token
  jsonPathExprs: $.access_token
  match_numbers: 0
  defaultValues: NOT_FOUND
</JSONPathExtractor>
```

Padrões de extração comuns:
| Campo | JSONPath |
|---|---|
| `access_token` | `$.access_token` ou `$.data.access_token` |
| `id` | `$.id` ou `$.data.id` |
| Genérico `{entidade}Id` | `$.{entidade}Id` ou `$.data.id` |

### Passo 7 — Adicionar Results Collector

Incluir um `SummaryReport` ResultCollector ao final do Thread Group com todos os flags de salvamento habilitados.

### Passo 8 — Validar a Estrutura

Antes de gerar a saída, verificar:
- [ ] O elemento raiz é `<jmeterTestPlan>`
- [ ] `TestPlan` → `hashTree` → `CSVDataSet`, `ConfigTestElement`, `HeaderManager`, `ThreadGroup`
- [ ] `ThreadGroup` → `hashTree` contém um `HTTPSamplerProxy` por endpoint na ordem de execução
- [ ] Cada `HTTPSamplerProxy` possui uma `hashTree` com pelo menos uma `ResponseAssertion`
- [ ] Endpoints que são dependências possuem `JSONPathExtractor` em sua `hashTree`
- [ ] O XML está bem formado (todas as tags corretamente fechadas)

### Passo 9 — Escrever o Arquivo

Escrever o XML gerado em:
- `tests/performance/dev/<scenario.file_prefix>.jmx`
- `tests/performance/hom/<scenario.file_prefix>.jmx`

Ambos os arquivos são idênticos nos Modos 1 e 2. Diferenças específicas de ambiente são tratadas via arquivo `.properties`.

Para o Modo 3, escrever apenas no caminho especificado pelo usuário (sem a divisão `dev/hom`).

---

## Exemplo de Nome de Arquivo de Saída

| Modo | Cenário | Arquivo de Saída |
|---|---|---|
| Pipeline (Modo 1 ou 2) | `default` | `tests/performance/dev/default.jmx` |
| Temporário (Modo 3) | `checkout_stress` | `tests/performance/dev/checkout_stress.jmx` |

---

## Observações

- Usar a sintaxe de propriedades JMeter `${variable}` — nunca hardcodar valores
- Os paths devem começar com `/`
- Não incluir `http://` ou `https://` nos paths dos samplers — usar os Padrões de Requisição HTTP para isso
- Usar codificação UTF-8 em todo o arquivo
- `ThreadGroup.scheduler=true` habilita a sustentação baseada em duração; `LoopController.loops=-1` permite iterações ilimitadas dentro da janela de sustentação
