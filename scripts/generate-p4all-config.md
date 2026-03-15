# Script: generate-p4all-config

**Modo:** Apenas Modo 1 (Pipeline Codebase) e Modo 2 (Pipeline Contratos de Gateway)
**Entrada:** Objeto de cenário de `generate-test-scenario.md`
**Saída:** `p4all.yml` — configuração de execução via pipeline para o framework p4all

---

## Objetivo

Gerar o arquivo de configuração `p4all.yml` que instrui o pipeline CI/CD sobre como executar o teste de performance JMeter e quais quality gates aplicar.

Este script **NÃO** é executado no Modo 3 (ambiente EC2 temporário).

---

## Procedimento

### Passo 1 — Construir a Seção de Execução

Mapear os parâmetros do cenário para os campos de execução do p4all:

| Campo p4all | Origem | Restrições |
|---|---|---|
| `concurrency` | `scenario.execution.concurrency` | Máximo: 50 |
| `ramp-up` | `scenario.execution.ramp_up_seconds` como `Ns` ou `Nm` | Máximo: 2m |
| `hold-for` | `scenario.execution.hold_seconds` como `Ns` ou `Nm` | Máximo: 10m |
| `scenario` | `scenario.name` | Fixo: `default` |

Regras de formato de duração:
- Se valor < 60 → usar `{valor}s` (ex.: `30s`, `45s`)
- Se valor ≥ 60 → usar `{valor/60}m` (ex.: `2m`, `5m`)

### Passo 2 — Construir a Seção de Cenários

```yaml
scenarios:
  <scenario.name>:
    script: <scenario.file_prefix>.jmx
```

### Passo 3 — Construir a Seção de Limites p4all

Aplicar os limites padrão (os mesmos para dev e hom):

```yaml
p4all:
  limit:
    rps: 300
    error-percent: 4
    time:
      p90: 500
      avg: 500
```

Esses limites impõem:
- Máximo de 300 requisições/segundo
- Taxa de erro máxima de 4%
- Tempo de resposta P90 ≤ 500ms
- Tempo médio de resposta ≤ 500ms

### Passo 4 — Validar as Restrições

Antes de escrever, verificar:
- [ ] `concurrency` ≤ 50
- [ ] Total de segundos de `ramp-up` ≤ 120 (2 minutos)
- [ ] Total de segundos de `hold-for` ≤ 600 (10 minutos)
- [ ] O nome de `scenario` corresponde a uma chave em `scenarios`
- [ ] O nome do arquivo `script` corresponde ao arquivo `.jmx` gerado

Se alguma restrição for violada, limitar ao valor máximo permitido e adicionar um comentário no arquivo gerado explicando o ajuste.

### Passo 5 — Escrever os Arquivos

Escrever em:
- `tests/performance/dev/p4all.yml`
- `tests/performance/hom/p4all.yml`

Ambos os arquivos são idênticos em conteúdo para o cenário padrão de pipeline.

---

## Formato de Saída

```yaml
# Configuração de Pipeline p4all
# Cenário: <scenario.name>
# Ambiente: <dev|hom>
# Gerado pela skill test-performance

execution:
  # Número de usuários virtuais simultâneos (máximo: 50)
  concurrency: 10

  # Duração para atingir a concorrência máxima (máximo: 2m)
  ramp-up: 5s

  # Duração da sustentação na concorrência máxima (máximo: 10m)
  hold-for: 30s

  # Cenário a executar (deve corresponder a uma chave em 'scenarios')
  scenario: default

scenarios:
  default:
    # Caminho para o script JMeter (relativo a este arquivo)
    script: default.jmx

p4all:
  limit:
    # Máximo de requisições por segundo permitido antes de reprovar o teste
    rps: 300

    # Percentual máximo de erros permitido (%)
    error-percent: 4

    # Limites de tempo de resposta em milissegundos
    time:
      # O tempo de resposta no percentil 90 deve estar abaixo deste valor
      p90: 500

      # O tempo médio de resposta deve estar abaixo deste valor
      avg: 500
```

---

## Observações

- Gerar este arquivo apenas para os modos pipeline (Modo 1 e Modo 2)
- A chave `scenario` em `execution` e a chave em `scenarios` devem corresponder exatamente
- O caminho do `script` é relativo à localização do arquivo `p4all.yml` (ambos ficam no mesmo diretório)
- Não adicionar funcionalidades opcionais do p4all (reporting, notificações) a menos que explicitamente solicitado pelo usuário
- Os valores de limite são padrões definidos pela skill `test-performance` e não devem ser alterados sem uma razão válida
