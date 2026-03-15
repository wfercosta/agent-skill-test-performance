# Estratégias de Teste de Performance

Este documento define as três estratégias de teste de performance suportadas pela skill `test-performance`.

---

## Seleção de Estratégia

| Modo | Estratégias Disponíveis |
|---|---|
| Modo 1 — Codebase (Pipeline) | Apenas Load Test Sustentado |
| Modo 2 — Contratos de Gateway (Pipeline) | Apenas Load Test Sustentado |
| Modo 3 — Ambiente EC2 Temporário | `load`, `stress`, `endurance` |

---

## 1. Load Test (`load`)

**Objetivo:** Validar o comportamento do sistema sob a carga nominal de produção esperada.

**Perfil:**
- Ramp-up gradual até a carga nominal
- Sustentação na carga nominal
- Ramp-down gradual

**Quando usar:** Validação de performance de linha de base, verificações de regressão antes de releases em produção.

**Parâmetros do Thread Group no JMeter:**

| Parâmetro | Orientação |
|---|---|
| `num_threads` | Número de usuários virtuais simultâneos na carga nominal |
| `ramp_time` | Duração para atingir a concorrência máxima (tipicamente 10–20% da duração total) |
| `duration` | Duração da sustentação na carga de pico |

**Perfil típico:**
```
Usuários
  ^
  |        ___________
  |       /           \
  |      /             \
  |_____/               \____
  +--------------------------> Tempo
    ramp-up  hold  ramp-down
```

---

## 2. Stress Test (`stress`)

**Objetivo:** Identificar o ponto de ruptura do sistema aumentando progressivamente a carga além da capacidade nominal.

**Perfil:**
- Ramp-up até a carga nominal
- Continuar aumentando até a carga de pico/stress
- Curta sustentação no pico
- Ramp-down

**Quando usar:** Planejamento de capacidade, identificação de gargalos, dimensionamento de infraestrutura.

**Parâmetros do Thread Group no JMeter:**

| Parâmetro | Orientação |
|---|---|
| `num_threads` | Usuários simultâneos de pico (acima do nominal — tipicamente 2x–3x) |
| `ramp_time` | Ramp-up mais longo para aumentar a pressão gradualmente |
| `duration` | Curta sustentação no pico (observe o limiar de falha) |

**Perfil típico:**
```
Usuários
  ^
  |              /\
  |             /  \
  |            /    \
  |           /      \
  |__________/        \______
  +--------------------------> Tempo
       ramp-up  hold  ramp-down
```

---

## 3. Endurance Test (`endurance`)

**Objetivo:** Detectar vazamentos de memória, esgotamento de recursos e degradação ao longo de execuções prolongadas com carga moderada.

**Perfil:**
- Ramp-up moderado
- Sustentação prolongada em carga sustentável (horas)
- Ramp-down gradual

**Quando usar:** Testes de estabilidade, detecção de vazamento de memória, esgotamento de pool de conexões, análise de garbage collection.

**Parâmetros do Thread Group no JMeter:**

| Parâmetro | Orientação |
|---|---|
| `num_threads` | Usuários simultâneos moderados (70–80% do nominal) |
| `ramp_time` | Ramp-up suave |
| `duration` | Sustentação longa (30 minutos a várias horas) |

**Perfil típico:**
```
Usuários
  ^
  |     ___________________________
  |    /                           \
  |   /                             \
  |__/                               \__
  +-------------------------------------> Tempo
   ramp   <--- sustentação prolongada --->  ramp-down
```

---

## Load Test Sustentado (Padrão Pipeline)

Usado automaticamente nos Modos 1 e 2. Configuração fixa baseada em `p4all-template.yml`.

**Perfil:**
- `ramp-up`: 5s
- `hold-for`: 30s
- `concurrency`: 10

**Restrições:**
- `concurrency` ≤ 50
- `ramp-up` ≤ 2m
- `hold-for` ≤ 10m
- Duração total ≤ 10 minutos

---

## Convenção de Nomenclatura de Arquivos

Os nomes dos arquivos de cenário combinam o nome fornecido pelo usuário e a estratégia:

| Nome do Cenário | Estratégia | Prefixo do Arquivo |
|---|---|---|
| checkout | stress | `checkout_stress` |
| login | load | `login_load` |
| payment | endurance | `payment_endurance` |

Arquivos gerados:
```
<nome>_<estrategia>.jmx
<nome>_<estrategia>.csv
<nome>_<estrategia>.properties
<nome>_<estrategia>_graph.yml
```
