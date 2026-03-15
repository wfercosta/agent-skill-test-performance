# Skill: test-performance

Você é um especialista em **Testes de Performance, Engenharia de Testes e Engenharia de Software com IA**.

Esta skill permite que agentes de IA criem e mantenham **testes de performance automatizados baseados em JMeter**, integrados com pipelines CI/CD ou executados em ambientes temporários.

Esta skill é **agnóstica de agente** e funciona corretamente quando usada em:
- GitHub Copilot Agents
- Devin
- OpenAI Codex Agents
- Claude Code
- Gemini Agents

---

## Capacidades

- Analisar código-fonte da aplicação e detectar APIs expostas
- Analisar arquivos de contrato OpenAPI/Swagger de um API Gateway
- Gerar scripts JMeter completos (`.jmx`)
- Gerar arquivos de massa de dados (`.csv`)
- Gerar arquivos de configuração de ambiente (`.properties`)
- Gerar grafos de sequência de execução (`.yml`)
- Gerar arquivos de configuração de pipeline p4all (`p4all.yml`)
- Suportar três modos de teste: pipeline (codebase), pipeline (contratos) e ambiente temporário EC2

---

## Comandos Suportados

| Comando | Descrição |
|---|---|
| `generate performance tests from codebase` | Analisa o diretório `/app`, detecta APIs e gera testes para os ambientes `dev` e `hom` |
| `generate performance tests from gateway contracts` | Analisa arquivos OpenAPI em `/contratos/*.yaml` e gera testes para os ambientes `dev` e `hom` |
| `generate temporary performance scenario` | Cria interativamente um cenário de performance customizado para ambiente EC2 temporário |

---

## Fluxo de Execução

### Modo 1 — Codebase (Pipeline)

1. Executar `scripts/analyze-app-apis.md` em `/app`
2. Executar `scripts/generate-test-scenario.md` → nome do cenário: `default`
3. Executar `scripts/generate-jmeter-script.md` → `default.jmx`
4. Executar `scripts/generate-test-data.md` → `default.csv`
5. Executar `scripts/generate-env-properties.md` → `default.properties`
6. Executar `scripts/generate-sequence-graph.md` → `default_graph.yml`
7. Executar `scripts/generate-p4all-config.md` → `p4all.yml`
8. Replicar artefatos para `tests/performance/dev/` e `tests/performance/hom/`

### Modo 2 — Contratos de Gateway (Pipeline)

1. Executar `scripts/analyze-gateway-contracts.md` em `/contratos/*.yaml`
2. Seguir os mesmos passos 2–8 do Modo 1

### Modo 3 — Ambiente Temporário (EC2)

1. Fazer as perguntas obrigatórias (ver abaixo)
2. Executar `scripts/generate-test-scenario.md` → nome do cenário: `[nome]_[estrategia]`
3. Executar `scripts/generate-jmeter-script.md` → `[nome]_[estrategia].jmx`
4. Executar `scripts/generate-test-data.md` → `[nome]_[estrategia].csv`
5. Executar `scripts/generate-env-properties.md` → `[nome]_[estrategia].properties`
6. Executar `scripts/generate-sequence-graph.md` → `[nome]_[estrategia]_graph.yml`
7. Neste modo, o arquivo `p4all.yml` **não** é gerado

#### Perguntas Obrigatórias para o Modo 3

Antes de gerar os artefatos, o agente **deve** perguntar ao usuário:
1. Qual o nome do cenário?
2. Qual a estratégia de teste? (`load`, `stress` ou `endurance`)
3. Número de usuários virtuais?
4. Carga nominal (requisições/segundo)?
5. Carga de pico (requisições/segundo)?
6. Duração do ramp-up?
7. Duração da sustentação?
8. Duração do ramp-down?

---

## Estrutura de Saída

```
tests/
└── performance/
    ├── dev/
    │   ├── <scenario_name>.jmx
    │   ├── <scenario_name>.csv
    │   ├── <scenario_name>.properties
    │   ├── <scenario_name>_graph.yml
    │   └── p4all.yml            ← Apenas modos pipeline
    └── hom/
        ├── <scenario_name>.jmx
        ├── <scenario_name>.csv
        ├── <scenario_name>.properties
        ├── <scenario_name>_graph.yml
        └── p4all.yml            ← Apenas modos pipeline
```

---

## Regras de Geração de Artefatos

### Script JMeter (`.jmx`)
- Deve ser um arquivo XML JMeter válido
- Deve usar CSV Data Set Config para a massa de dados
- Deve usar HTTP Request samplers para cada endpoint da API
- Deve usar Response Assertion para validação básica
- Deve configurar o Thread Group com ramp-up/hold/ramp-down a partir dos parâmetros do cenário
- Respeitar a ordem de execução derivada do grafo de dependências entre APIs

### Massa de Dados (`.csv`)
- Linha de cabeçalho seguida de linhas de dados de exemplo
- Campos mínimos: `user_id`, `account_id`, `product_id`
- Adaptar campos às APIs detectadas (ex.: incluir `product_id` se existir uma API de produtos)
- Incluir pelo menos 5 linhas de dados de exemplo

### Propriedades de Ambiente (`.properties`)
- Placeholders com comentários descritivos
- Campos mínimos: `base_url`, `auth_token`, `client_id`
- Adaptar conforme as necessidades de autenticação e configuração detectadas

### Grafo de Sequência (`_graph.yml`)
- `nodes`: lista de nomes de APIs na ordem lógica de execução
- `edges`: dependências direcionadas entre APIs

### Configuração p4all (`p4all.yml`)
- Apenas modos pipeline
- `execution.concurrency` máximo: 50
- `execution.ramp-up` máximo: 2m
- `execution.hold-for` máximo: 10m
- `p4all.limit.rps`: 300
- `p4all.limit.error-percent`: 4
- `p4all.limit.time.p90`: 500ms
- `p4all.limit.time.avg`: 500ms

---

## Referências

| Arquivo | Finalidade |
|---|---|
| `references/jmeter-template.md` | Estrutura XML do JMeter e padrões de sampler |
| `references/p4all-template.yml` | Template de configuração p4all |
| `references/performance-strategies.md` | Definições das estratégias load, stress e endurance |
| `references/graph-template.yml` | Formato do grafo de sequência |
| `references/env-template.properties` | Template de propriedades de ambiente |

---

## Scripts

| Arquivo | Finalidade |
|---|---|
| `scripts/analyze-app-apis.md` | Detectar APIs a partir do código-fonte da aplicação |
| `scripts/analyze-gateway-contracts.md` | Detectar APIs a partir de arquivos de contrato OpenAPI |
| `scripts/generate-test-scenario.md` | Definir parâmetros do cenário |
| `scripts/generate-jmeter-script.md` | Gerar arquivo `.jmx` |
| `scripts/generate-test-data.md` | Gerar arquivo `.csv` |
| `scripts/generate-sequence-graph.md` | Gerar arquivo `_graph.yml` |
| `scripts/generate-env-properties.md` | Gerar arquivo `.properties` |
| `scripts/generate-p4all-config.md` | Gerar arquivo `p4all.yml` |
