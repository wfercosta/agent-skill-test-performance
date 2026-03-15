# test-performance

Skill para geração automatizada de testes de performance baseados em JMeter, integrada com pipelines CI/CD ou executável em ambientes temporários.

---

## Visão Geral

A skill `test-performance` permite que agentes de IA analisem uma aplicação ou contratos de API e gerem, de forma autônoma, todos os artefatos necessários para executar um teste de performance: script JMeter, massa de dados, configuração de ambiente, grafo de sequência e configuração de pipeline.

É **agnóstica de agente** e funciona em GitHub Copilot Agents, Devin, OpenAI Codex Agents, Claude Code e Gemini Agents.

---

## Estrutura

```
test-performance/
├── SKILL.md                          # Documento principal da skill
├── references/
│   ├── jmeter-template.md            # Estrutura XML JMeter e padrões de sampler
│   ├── p4all-template.yml            # Template de configuração p4all
│   ├── performance-strategies.md     # Definições das estratégias de teste
│   ├── graph-template.yml            # Formato do grafo de sequência
│   └── env-template.properties       # Template de propriedades de ambiente
└── scripts/
    ├── analyze-app-apis.md           # Detecta APIs no código-fonte (/app)
    ├── analyze-gateway-contracts.md  # Parseia contratos OpenAPI (/contratos)
    ├── generate-test-scenario.md     # Consolida parâmetros do cenário
    ├── generate-jmeter-script.md     # Gera o arquivo .jmx
    ├── generate-test-data.md         # Gera a massa de dados .csv
    ├── generate-sequence-graph.md    # Gera o grafo de dependências _graph.yml
    ├── generate-env-properties.md    # Gera o arquivo .properties por ambiente
    └── generate-p4all-config.md      # Gera o p4all.yml para pipeline
```

---

## Modos de Uso

### Modo 1 — A partir do Código-fonte (Pipeline)

Analisa o diretório `/app`, detecta APIs expostas e gera os artefatos de teste.

```
gerar testes de performance a partir do codigo
```

Estrutura de projeto esperada:
```
/
├── app/
└── tests/
```

### Modo 2 — A partir de Contratos de API Gateway (Pipeline)

Analisa arquivos OpenAPI em `/contratos` e gera os artefatos de teste.

```
gerar testes de performance a partir dos contratos
```

Estrutura de projeto esperada:
```
/
├── contratos/
│   ├── api1.yaml
│   └── api2.yaml
└── tests/
```

### Modo 3 — Cenário Temporário (EC2)

Cria um cenário customizado de forma interativa, sem execução via pipeline. O agente fará as seguintes perguntas:

- Nome do cenário
- Estratégia: `load`, `stress` ou `endurance`
- Número de usuários virtuais
- Carga nominal e de pico (req/s)
- Durações de ramp-up, sustentação e ramp-down

```
gerar cenario de performance temporario
```

---

## Artefatos Gerados

Para cada ambiente (`dev` e `hom`), a skill gera:

```
tests/
└── performance/
    ├── dev/
    │   ├── <cenario>.jmx             # Script JMeter
    │   ├── <cenario>.csv             # Massa de dados
    │   ├── <cenario>.properties      # Configuração de ambiente (preencher antes de executar)
    │   ├── <cenario>_graph.yml       # Grafo de sequência das APIs
    │   └── p4all.yml                 # Configuração de pipeline (Modos 1 e 2)
    └── hom/
        └── ...                       # Mesma estrutura
```

Nos Modos 1 e 2, o nome do cenário é sempre `default`. No Modo 3, o nome segue o padrão `[nome]_[estrategia]` (ex.: `checkout_stress`, `login_load`).

---

## Estratégias de Teste

| Estratégia | Objetivo | Disponível em |
|---|---|---|
| **Load** | Validar comportamento sob carga nominal de produção | Modo 3 |
| **Stress** | Identificar o ponto de ruptura do sistema | Modo 3 |
| **Endurance** | Detectar degradação em execuções prolongadas | Modo 3 |
| **Sustained Load** | Carga sustentada padrão com limites fixos | Modos 1 e 2 |

---

## Limites do Pipeline (Modos 1 e 2)

| Parâmetro | Valor |
|---|---|
| Concorrência máxima | 50 usuários virtuais |
| Ramp-up máximo | 2 minutos |
| Hold-for máximo | 10 minutos |
| RPS máximo permitido | 300 req/s |
| Taxa de erro máxima | 4% |
| Tempo de resposta P90 | ≤ 500ms |
| Tempo médio de resposta | ≤ 500ms |

---

## Como Usar

1. Copie este diretório `test-performance/` para o repositório do projeto alvo ou para o diretório de skills do seu agente.
2. Invoque um dos comandos suportados no chat com o agente.
3. O agente executará os scripts na ordem correta e gerará os artefatos em `tests/performance/`.
4. Preencha os arquivos `.properties` com os valores reais de cada ambiente antes de executar os testes.
