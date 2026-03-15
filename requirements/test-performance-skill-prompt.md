# Agent Skill Generator -- test-performance

Você é um especialista em **Performance Testing, Test Engineering e AI
Software Engineering**.

Seu trabalho é gerar uma **Agent Skill chamada `test-performance`**.

Essa habilidade permite que agentes de IA criem e mantenham **testes de
performance automatizados baseados em JMeter**, integrados com pipelines
CI/CD ou executados em ambientes temporários.

A skill deve ser **agnóstica de agente**, funcionando corretamente
quando usada em:

-   GitHub Copilot Agents
-   Devin
-   OpenAI Codex Agents
-   Claude Code
-   Gemini Agents

------------------------------------------------------------------------

# Estrutura da Skill

    test-performance/
    ├── SKILL.md
    ├── references/
    └── scripts/

## SKILL.md

Documento principal descrevendo:

-   capacidades da skill
-   comandos suportados
-   fluxo de execução
-   regras de geração dos artefatos

## references/

Contém templates e conhecimentos reutilizáveis.

    references/
    ├── jmeter-template.md
    ├── p4all-template.yml
    ├── performance-strategies.md
    ├── graph-template.yml
    └── env-template.properties

## scripts/

Contém procedimentos executáveis pela skill.

    scripts/
    ├── analyze-app-apis.md
    ├── analyze-gateway-contracts.md
    ├── generate-test-scenario.md
    ├── generate-jmeter-script.md
    ├── generate-test-data.md
    ├── generate-sequence-graph.md
    ├── generate-env-properties.md
    └── generate-p4all-config.md

------------------------------------------------------------------------

# Estrutura de projetos suportados

## Projeto com aplicação

    /
    ├── app/
    ├── infra/
    └── tests/

A skill deve analisar APIs dentro de:

    /app

## Projeto baseado em contratos

    /
    ├── contratos/
    │   ├── api1.yaml
    │   ├── api2.yaml
    │   └── api3.yaml
    └── tests/

A skill deve analisar arquivos **OpenAPI** dentro de:

    /contratos

------------------------------------------------------------------------

# Estrutura de saída gerada pela skill

    tests/
    └── performance/
        ├── dev/
        └── hom/

Cada ambiente deve possuir:

    p4all.yml
    <scenario_name>.jmx
    <scenario_name>.csv
    <scenario_name>.properties
    <scenario_name>_graph.yml

------------------------------------------------------------------------

# Modalidades da Skill

## 1. Testes baseados na Codebase (Pipeline)

O agente deve:

1.  Analisar o diretório `/app`
2.  Detectar APIs expostas
3.  Determinar dependências entre APIs
4.  Gerar script JMeter, massa de dados e variáveis
5.  Criar estrutura em:

```{=html}
<!-- -->
```
    tests/performance/dev/
    tests/performance/hom/

### Estratégia

Sempre utilizar **Load Test Sustentado** com:

-   ramp-up
-   hold
-   ramp-down

Tempo máximo: **10 minutos**

### Cenário

Nome fixo:

    default

Arquivos:

    default.jmx
    default.csv
    default.properties
    default_graph.yml

### Configuração p4all.yml

    execution:
      concurrency: 10
      ramp-up: 5s
      hold-for: 30s
      scenario: default

    scenarios:
      default:
        script: default.jmx

    p4all:
      limit:
        rps: 300
        error-percent: 4
        time:
          p90: 500
          avg: 500

Regras:

-   concurrency máximo: 50
-   ramp-up máximo: 2m
-   hold-for máximo: 10m

------------------------------------------------------------------------

## 2. Testes baseados em contratos de API Gateway (Pipeline)

O agente deve analisar:

    /contratos/*.yaml

Extrair endpoints, dependências e gerar:

    tests/performance/dev/
    tests/performance/hom/

Cenário também chamado:

    default

------------------------------------------------------------------------

## 3. Testes em ambiente temporário (EC2)

Nesta modalidade não há execução via pipeline.

O agente deve gerar cenários customizados.

### Estratégias

Usuário pode escolher:

-   load
-   stress
-   endurance

### Perguntas obrigatórias

O agente deve perguntar:

-   número de usuários virtuais
-   carga nominal
-   carga de pico
-   duração ramp-up
-   duração sustentação
-   duração ramp-down

### Nome do cenário

Perguntar ao usuário:

    Qual o nome do cenário?

Formato final:

    [nome_cenario]_[estrategia]

Exemplo:

    checkout_stress
    login_load
    payment_endurance

Arquivos gerados:

    checkout_stress.jmx
    checkout_stress.csv
    checkout_stress.properties
    checkout_stress_graph.yml

------------------------------------------------------------------------

# Estrutura final

    tests/
    └── performance
        ├── dev
        │   ├── scenario.jmx
        │   ├── scenario.csv
        │   ├── scenario.properties
        │   └── scenario_graph.yml
        └── hom
            ├── scenario.jmx
            ├── scenario.csv
            ├── scenario.properties
            └── scenario_graph.yml

------------------------------------------------------------------------

# Arquivo properties

Gerar placeholders com comentários:

    base_url=
    auth_token=
    client_id=

------------------------------------------------------------------------

# Arquivo CSV

Massa de dados exemplo:

    user_id
    account_id
    product_id

------------------------------------------------------------------------

# Arquivo graph.yml

Sequência de execução das APIs.

    nodes:
      - login
      - get-products
      - checkout

    edges:
      - from: login
        to: get-products
      - from: get-products
        to: checkout

------------------------------------------------------------------------

# Regras do agente

O agente deve:

1.  analisar código ou contratos
2.  identificar dependências entre APIs
3.  gerar sequência lógica de execução
4.  criar massa de dados
5.  gerar scripts JMeter válidos
6.  gerar configurações consistentes

------------------------------------------------------------------------

# Comandos suportados

Usuário pode invocar:

    generate performance tests from codebase
    generate performance tests from gateway contracts
    generate temporary performance scenario

------------------------------------------------------------------------

# Objetivo final

Gerar **testes de performance reproduzíveis, consistentes e integrados à
pipeline**, ou executáveis em ambientes temporários para experimentação
de carga.
