# Script: generate-test-scenario

**Entrada:** Saída da análise de `analyze-app-apis.md` ou `analyze-gateway-contracts.md`, mais o modo e as entradas do usuário
**Saída:** Objeto de parâmetros do cenário usado por todos os scripts de geração subsequentes

---

## Objetivo

Consolidar todas as entradas em um objeto de configuração de cenário unificado que orienta a geração dos arquivos `.jmx`, `.csv`, `.properties`, `_graph.yml` e `p4all.yml`.

---

## Procedimento

### Modo 1 — Codebase (Pipeline) e Modo 2 — Contratos de Gateway (Pipeline)

Nenhuma interação com o usuário é necessária. Aplicar os valores padrão fixos:

```yaml
scenario:
  name: default
  file_prefix: default
  mode: pipeline
  strategy: sustained-load

  execution:
    concurrency: 10
    ramp_up_seconds: 5
    hold_seconds: 30
    ramp_down_seconds: 5

  limits:
    rps: 300
    error_percent: 4
    p90_ms: 500
    avg_ms: 500

  endpoints: <lista do passo de análise>
  execution_order: <ordenação topológica do passo de análise>
  csv_headers: <derivado dos campos de dados dos endpoints>
```

### Modo 3 — Ambiente EC2 Temporário

Coletar parâmetros a partir das respostas do usuário:

#### 3.1 — Coletar Nome do Cenário

Prompt:
```
Qual o nome do cenário? (use minúsculas, sem espaços, ex.: checkout, login, payment)
```

Validar: apenas letras minúsculas, dígitos e hífens. Sem espaços.

#### 3.2 — Coletar Estratégia

Prompt:
```
Qual a estratégia de teste?
  1. load      — carga nominal de produção
  2. stress    — acima do nominal, encontrar ponto de ruptura
  3. endurance — duração prolongada, teste de estabilidade
```

Aceitar: `load`, `stress` ou `endurance`.

#### 3.3 — Derivar o Prefixo do Arquivo

```
file_prefix = {nome_cenario}_{estrategia}
```

Exemplos:
- `checkout_stress`
- `login_load`
- `payment_endurance`

#### 3.4 — Coletar Parâmetros de Execução

Solicitar cada um:

```
Número de usuários virtuais (threads simultâneas)?
Meta de carga nominal (requisições/segundo)?
Meta de carga de pico (requisições/segundo)?
Duração do ramp-up? (formato: 30s, 2m, 5m)
Duração da sustentação? (formato: 30s, 5m, 30m)
Duração do ramp-down? (formato: 30s, 2m, 5m)
```

Converter todas as durações para segundos para compatibilidade com o JMeter:
- `30s` → 30
- `2m` → 120
- `5m` → 300

#### 3.5 — Construir o Objeto de Cenário

```yaml
scenario:
  name: <entrada do usuário>
  file_prefix: <nome>_<estrategia>
  mode: temporary
  strategy: <load | stress | endurance>

  execution:
    concurrency: <entrada do usuário>
    ramp_up_seconds: <convertido>
    hold_seconds: <convertido>
    ramp_down_seconds: <convertido>
    nominal_rps: <entrada do usuário>
    peak_rps: <entrada do usuário>

  endpoints: <lista do passo de análise>
  execution_order: <ordenação topológica do passo de análise>
  csv_headers: <derivado dos campos de dados dos endpoints>
```

---

## Derivação dos Cabeçalhos CSV

Coletar todos os campos de dados únicos necessários em todos os endpoints:

1. Sempre incluir: `user_id`, `account_id`
2. Para cada endpoint com parâmetros de path, adicionar uma coluna com o valor esperado
3. Para cada endpoint POST/PUT, adicionar colunas para campos do corpo ainda não incluídos
4. Para cada extração de resposta necessária para encadeamento, adicionar uma coluna (esses valores serão populados em tempo de execução pelos extractors do JMeter — incluí-los no CSV com valores placeholder)

Exemplo de derivação:
```
Endpoints usam: {userId}, {productId}, {orderId}
Corpo do POST /cart: { quantity }
Extrações de resposta: access_token, cartId

Cabeçalhos CSV: user_id, account_id, product_id, quantity
(access_token e cartId são extrações em tempo de execução — não entram no CSV)
```

---

## Saída

O objeto de cenário é repassado como contexto para:
- `generate-jmeter-script.md`
- `generate-test-data.md`
- `generate-env-properties.md`
- `generate-sequence-graph.md`
- `generate-p4all-config.md` (apenas modos pipeline)
