# Script: generate-sequence-graph

**Entrada:** Objeto de cenário de `generate-test-scenario.md` (especificamente `execution_order` e `endpoints`)
**Saída:** `<scenario.file_prefix>_graph.yml` — grafo de sequência de execução das APIs

---

## Objetivo

Gerar um arquivo YAML que documenta a sequência lógica de execução das chamadas de API no cenário de teste. Este grafo representa as relações de dependência direcionadas entre endpoints e serve como documentação e referência para entender o fluxo do teste.

---

## Procedimento

### Passo 1 — Extrair os Nós

Cada nó é uma etapa de API nomeada, derivada de `scenario.execution_order`.

Usar o campo `name` de cada endpoint (kebab-case).

### Passo 2 — Extrair as Arestas

Para cada endpoint na lista `endpoints`, para cada item em sua lista `depends_on`, criar uma aresta direcionada:

```
edge: { from: <nome_da_dependência>, to: <nome_deste_endpoint> }
```

### Passo 3 — Validar o Grafo

Antes de escrever:
- [ ] Todos os nomes de nós referenciados em `edges` existem em `nodes`
- [ ] Sem nomes de nós duplicados
- [ ] Sem arestas duplicadas
- [ ] Sem ciclos (o grafo deve ser um DAG — Directed Acyclic Graph)
- [ ] Os nós estão listados em ordem topológica (dependências antes dos dependentes)

### Passo 4 — Escrever o Arquivo

Escrever o YAML em:
- `tests/performance/dev/<scenario.file_prefix>_graph.yml`
- `tests/performance/hom/<scenario.file_prefix>_graph.yml`

---

## Formato de Saída

```yaml
# Grafo de sequência para o cenário: <scenario.name>
# Gerado pela skill test-performance
# Este grafo representa a ordem de execução e as dependências entre as APIs.

nodes:
  - <nome_etapa_1>
  - <nome_etapa_2>
  - <nome_etapa_3>

edges:
  - from: <nome_etapa_1>
    to: <nome_etapa_2>
  - from: <nome_etapa_2>
    to: <nome_etapa_3>
```

---

## Exemplos

### Cadeia linear (authenticate → get-products → checkout)

```yaml
# Grafo de sequência para o cenário: default
# Gerado pela skill test-performance

nodes:
  - authenticate
  - get-products
  - checkout

edges:
  - from: authenticate
    to: get-products
  - from: get-products
    to: checkout
```

### Ramificação (caminhos paralelos após autenticação)

```yaml
# Grafo de sequência para o cenário: full-flow

nodes:
  - authenticate
  - get-user-profile
  - get-catalog
  - get-cart
  - add-to-cart
  - place-order

edges:
  - from: authenticate
    to: get-user-profile
  - from: authenticate
    to: get-catalog
  - from: authenticate
    to: get-cart
  - from: get-catalog
    to: add-to-cart
  - from: get-cart
    to: add-to-cart
  - from: add-to-cart
    to: place-order
```

### Endpoint único (sem dependências)

```yaml
# Grafo de sequência para o cenário: health-check-load

nodes:
  - get-health

edges: []
```

---

## Observações

- Os nomes dos nós devem usar kebab-case (minúsculas, hífens como separadores)
- A ordem da lista `nodes` deve corresponder à ordem topológica de execução
- Se não houver dependências entre os endpoints, `edges` é uma lista vazia `[]`
- Este arquivo é informativo e não afeta diretamente a execução do JMeter — a ordem de execução está codificada na sequência de samplers do `.jmx`
