# Script: generate-env-properties

**Entrada:** Objeto de cenário de `generate-test-scenario.md`, saída da análise (esquemas de autenticação detectados, URLs base)
**Saída:** `<scenario.file_prefix>.properties` — configuração de runtime específica por ambiente

---

## Objetivo

Gerar um arquivo `.properties` para cada ambiente (`dev`, `hom`) contendo os valores de configuração de runtime exigidos pelo teste JMeter. Esses arquivos armazenam configurações específicas por ambiente e devem ser preenchidos antes da execução dos testes.

---

## Procedimento

### Passo 1 — Determinar as Propriedades Base

Sempre incluir as seguintes propriedades base:

```properties
base_url=
auth_token=
client_id=
```

### Passo 2 — Adaptar à Autenticação Detectada

Com base nos esquemas de autenticação detectados durante a análise das APIs:

| Tipo de Auth | Propriedades Adicionais |
|---|---|
| OAuth2 / Bearer Token | `auth_token=` (já incluída) |
| API Key no cabeçalho | `api_key=` |
| Basic Auth | `basic_auth_username=`, `basic_auth_password=` |
| Client Credentials | `client_id=`, `client_secret=` |
| Sem autenticação | Omitir propriedades de auth |

### Passo 3 — Adaptar à Configuração Detectada

Com base nas APIs detectadas e seus requisitos:

| Funcionalidade Detectada | Propriedades Adicionais |
|---|---|
| Versionamento de API (`/v1`, `/v2`) | `api_version=v1` |
| Roteamento por tenant/organização | `tenant_id=`, `org_id=` |
| Prefixo de roteamento customizado | `api_prefix=` |
| Configuração de timeout | `connect_timeout=5000`, `response_timeout=30000` |

### Passo 4 — Adicionar Marcador de Ambiente

Adicionar um marcador apenas como comentário para identificar o ambiente:

```properties
# Ambiente: dev
```
ou
```properties
# Ambiente: hom
```

### Passo 5 — Escrever os Arquivos

Escrever os arquivos específicos por ambiente em:
- `tests/performance/dev/<scenario.file_prefix>.properties`
- `tests/performance/hom/<scenario.file_prefix>.properties`

Os arquivos possuem as mesmas chaves de propriedade, mas valores diferentes (o arquivo `dev` usa endpoints de desenvolvimento, o arquivo `hom` usa endpoints de homologação).

Como os valores são placeholders, a estrutura é idêntica. O usuário deve preencher os valores corretos por ambiente antes de executar os testes.

---

## Formato de Saída

```properties
# Configuração de Ambiente para Teste de Performance
# Cenário: <scenario.name>
# Ambiente: <dev|hom>
# Gerado pela skill test-performance
#
# IMPORTANTE: Preencha todos os valores antes de executar o teste.
# NÃO commite valores sensíveis (tokens, senhas, secrets) no controle de versão.

# -----------------------------------------------------------------------
# URL Base
# A URL raiz da aplicação em teste (sem barra final).
# Exemplo (dev): https://api.myapp.dev.internal
# Exemplo (hom): https://api.myapp.hom.internal
# -----------------------------------------------------------------------
base_url=

# -----------------------------------------------------------------------
# Token de Autenticação
# Bearer token para autenticar as requisições da API.
# Obtenha via endpoint de autenticação antes de executar o teste,
# ou forneça um token de serviço de longa duração para pipelines automatizados.
# -----------------------------------------------------------------------
auth_token=

# -----------------------------------------------------------------------
# Client ID
# Identificador do cliente usado em fluxos OAuth2 ou roteamento do API Gateway.
# -----------------------------------------------------------------------
client_id=
```

---

## Exemplo: Cenário de E-commerce com OAuth2

```properties
# Configuração de Ambiente para Teste de Performance
# Cenário: checkout_stress
# Ambiente: dev
# Gerado pela skill test-performance

base_url=
auth_token=
client_id=
client_secret=
api_version=v1
connect_timeout=5000
response_timeout=30000
```

---

## Observações

- Todos os valores devem permanecer vazios (`chave=`) — são placeholders a serem preenchidos antes da execução do teste
- Os comentários devem explicar o que cada propriedade é e fornecer um exemplo concreto do formato do valor
- NÃO incluir credenciais, tokens ou senhas reais no arquivo gerado
- Os arquivos `dev` e `hom` são estruturalmente idênticos; apenas os comentários diferem para indicar o ambiente
- Os nomes das propriedades devem corresponder exatamente aos nomes de variáveis JMeter usados no script `.jmx` (ex.: `${base_url}`, `${auth_token}`)
