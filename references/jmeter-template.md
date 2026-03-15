# Template de Script JMeter

Este documento fornece a estrutura canônica e os padrões para geração de arquivos `.jmx` JMeter válidos dentro da skill `test-performance`.

---

## Estrutura XML Base

```xml
<?xml version="1.0" encoding="UTF-8"?>
<jmeterTestPlan version="1.2" properties="5.0" jmeter="5.6.3">
  <hashTree>
    <TestPlan guiclass="TestPlanGui" testclass="TestPlan" testname="${SCENARIO_NAME}" enabled="true">
      <stringProp name="TestPlan.comments">Gerado pela skill test-performance</stringProp>
      <boolProp name="TestPlan.functional_mode">false</boolProp>
      <boolProp name="TestPlan.tearDown_on_shutdown">true</boolProp>
      <boolProp name="TestPlan.serialize_threadgroups">false</boolProp>
      <elementProp name="TestPlan.user_defined_variables" elementType="Arguments" guiclass="ArgumentsPanel" testclass="Arguments" testname="Variáveis Definidas pelo Usuário" enabled="true">
        <collectionProp name="Arguments.arguments"/>
      </elementProp>
    </TestPlan>
    <hashTree>

      <!-- Configuração do CSV Data Set -->
      <CSVDataSet guiclass="TestBeanGUI" testclass="CSVDataSet" testname="Massa de Dados" enabled="true">
        <stringProp name="delimiter">,</stringProp>
        <stringProp name="fileEncoding">UTF-8</stringProp>
        <stringProp name="filename">${SCENARIO_NAME}.csv</stringProp>
        <boolProp name="ignoreFirstLine">true</boolProp>
        <boolProp name="quotedData">false</boolProp>
        <boolProp name="recycle">true</boolProp>
        <boolProp name="stopThread">false</boolProp>
        <stringProp name="variableNames">${CSV_HEADERS}</stringProp>
        <stringProp name="shareMode">shareMode.all</stringProp>
      </CSVDataSet>
      <hashTree/>

      <!-- Padrões de Requisição HTTP -->
      <ConfigTestElement guiclass="HttpDefaultsGui" testclass="ConfigTestElement" testname="Padrões de Requisição HTTP" enabled="true">
        <elementProp name="HTTPsampler.Arguments" elementType="Arguments" guiclass="HTTPArgumentsPanel" testclass="Arguments" testname="Variáveis Definidas pelo Usuário" enabled="true">
          <collectionProp name="Arguments.arguments"/>
        </elementProp>
        <stringProp name="HTTPSampler.domain">${base_url}</stringProp>
        <stringProp name="HTTPSampler.port"></stringProp>
        <stringProp name="HTTPSampler.protocol">https</stringProp>
        <stringProp name="HTTPSampler.contentEncoding">UTF-8</stringProp>
        <stringProp name="HTTPSampler.connect_timeout">5000</stringProp>
        <stringProp name="HTTPSampler.response_timeout">30000</stringProp>
      </ConfigTestElement>
      <hashTree/>

      <!-- Gerenciador de Cabeçalhos HTTP -->
      <HeaderManager guiclass="HeaderPanel" testclass="HeaderManager" testname="Gerenciador de Cabeçalhos HTTP" enabled="true">
        <collectionProp name="HeaderManager.headers">
          <elementProp name="" elementType="Header">
            <stringProp name="Header.name">Content-Type</stringProp>
            <stringProp name="Header.value">application/json</stringProp>
          </elementProp>
          <elementProp name="" elementType="Header">
            <stringProp name="Header.name">Authorization</stringProp>
            <stringProp name="Header.value">Bearer ${auth_token}</stringProp>
          </elementProp>
        </collectionProp>
      </HeaderManager>
      <hashTree/>

      <!-- Thread Group -->
      <ThreadGroup guiclass="ThreadGroupGui" testclass="ThreadGroup" testname="Usuários Virtuais" enabled="true">
        <stringProp name="ThreadGroup.on_sample_error">continue</stringProp>
        <elementProp name="ThreadGroup.main_controller" elementType="LoopController" guiclass="LoopControlPanel" testclass="LoopController" testname="Loop Controller" enabled="true">
          <boolProp name="LoopController.continue_forever">false</boolProp>
          <intProp name="LoopController.loops">-1</intProp>
        </elementProp>
        <stringProp name="ThreadGroup.num_threads">${CONCURRENCY}</stringProp>
        <stringProp name="ThreadGroup.ramp_time">${RAMP_UP_SECONDS}</stringProp>
        <boolProp name="ThreadGroup.scheduler">true</boolProp>
        <stringProp name="ThreadGroup.duration">${HOLD_SECONDS}</stringProp>
        <stringProp name="ThreadGroup.delay">0</stringProp>
        <boolProp name="ThreadGroup.same_user_on_next_iteration">true</boolProp>
      </ThreadGroup>
      <hashTree>

        <!-- Samplers HTTP (um por endpoint de API) -->
        ${HTTP_SAMPLERS}

        <!-- Resultados -->
        <ResultCollector guiclass="SummaryReport" testclass="ResultCollector" testname="Relatório Resumido" enabled="true">
          <boolProp name="ResultCollector.error_logging">false</boolProp>
          <objProp>
            <name>saveConfig</name>
            <value class="SampleSaveConfiguration">
              <time>true</time>
              <latency>true</latency>
              <timestamp>true</timestamp>
              <success>true</success>
              <label>true</label>
              <code>true</code>
              <message>true</message>
              <threadName>true</threadName>
              <dataType>true</dataType>
              <encoding>false</encoding>
              <assertions>true</assertions>
              <subresults>true</subresults>
              <responseData>false</responseData>
              <samplerData>false</samplerData>
              <xml>false</xml>
              <fieldNames>true</fieldNames>
              <responseHeaders>false</responseHeaders>
              <requestHeaders>false</requestHeaders>
              <responseDataOnError>false</responseDataOnError>
              <saveAssertionResultsFailureMessage>true</saveAssertionResultsFailureMessage>
              <assertionsResultsToSave>0</assertionsResultsToSave>
              <bytes>true</bytes>
              <sentBytes>true</sentBytes>
              <url>true</url>
              <threadCounts>true</threadCounts>
              <idleTime>true</idleTime>
              <connectTime>true</connectTime>
            </value>
          </objProp>
          <stringProp name="filename"></stringProp>
        </ResultCollector>
        <hashTree/>

      </hashTree>
    </hashTree>
  </hashTree>
</jmeterTestPlan>
```

---

## Padrão de Sampler HTTP (GET)

```xml
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="${ENDPOINT_NAME}" enabled="true">
  <elementProp name="HTTPsampler.Arguments" elementType="Arguments" guiclass="HTTPArgumentsPanel" testclass="Arguments" testname="Variáveis Definidas pelo Usuário" enabled="true">
    <collectionProp name="Arguments.arguments"/>
  </elementProp>
  <stringProp name="HTTPSampler.domain"></stringProp>
  <stringProp name="HTTPSampler.port"></stringProp>
  <stringProp name="HTTPSampler.protocol"></stringProp>
  <stringProp name="HTTPSampler.contentEncoding">UTF-8</stringProp>
  <stringProp name="HTTPSampler.path">${ENDPOINT_PATH}</stringProp>
  <stringProp name="HTTPSampler.method">GET</stringProp>
  <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
  <boolProp name="HTTPSampler.auto_redirects">false</boolProp>
  <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
  <boolProp name="HTTPSampler.DO_MULTIPART_POST">false</boolProp>
</HTTPSamplerProxy>
<hashTree>
  <ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" testname="Assertion de Código de Resposta" enabled="true">
    <collectionProp name="Asserion.test_strings">
      <stringProp name="49586">200</stringProp>
    </collectionProp>
    <stringProp name="Assertion.custom_message"></stringProp>
    <stringProp name="Assertion.test_field">Assertion.response_code</stringProp>
    <boolProp name="Assertion.assume_success">false</boolProp>
    <intProp name="Assertion.test_type">8</intProp>
  </ResponseAssertion>
  <hashTree/>
</hashTree>
```

---

## Padrão de Sampler HTTP (POST com corpo JSON)

```xml
<HTTPSamplerProxy guiclass="HttpTestSampleGui" testclass="HTTPSamplerProxy" testname="${ENDPOINT_NAME}" enabled="true">
  <boolProp name="HTTPSampler.postBodyRaw">true</boolProp>
  <elementProp name="HTTPsampler.Arguments" elementType="Arguments">
    <collectionProp name="Arguments.arguments">
      <elementProp name="" elementType="HTTPArgument">
        <boolProp name="HTTPArgument.always_encode">false</boolProp>
        <stringProp name="Argument.value">${REQUEST_BODY}</stringProp>
        <stringProp name="Argument.metadata">=</stringProp>
      </elementProp>
    </collectionProp>
  </elementProp>
  <stringProp name="HTTPSampler.domain"></stringProp>
  <stringProp name="HTTPSampler.port"></stringProp>
  <stringProp name="HTTPSampler.protocol"></stringProp>
  <stringProp name="HTTPSampler.contentEncoding">UTF-8</stringProp>
  <stringProp name="HTTPSampler.path">${ENDPOINT_PATH}</stringProp>
  <stringProp name="HTTPSampler.method">POST</stringProp>
  <boolProp name="HTTPSampler.follow_redirects">true</boolProp>
  <boolProp name="HTTPSampler.auto_redirects">false</boolProp>
  <boolProp name="HTTPSampler.use_keepalive">true</boolProp>
  <boolProp name="HTTPSampler.DO_MULTIPART_POST">false</boolProp>
</HTTPSamplerProxy>
<hashTree>
  <ResponseAssertion guiclass="AssertionGui" testclass="ResponseAssertion" testname="Assertion de Código de Resposta" enabled="true">
    <collectionProp name="Asserion.test_strings">
      <stringProp name="49586">200</stringProp>
      <stringProp name="49587">201</stringProp>
    </collectionProp>
    <stringProp name="Assertion.custom_message"></stringProp>
    <stringProp name="Assertion.test_field">Assertion.response_code</stringProp>
    <boolProp name="Assertion.assume_success">false</boolProp>
    <intProp name="Assertion.test_type">8</intProp>
  </ResponseAssertion>
  <hashTree/>
</hashTree>
```

---

## Padrão de JSON Extractor (para encadeamento de requisições)

Use este padrão para extrair valores de respostas e repassá-los às requisições seguintes.

```xml
<JSONPathExtractor guiclass="JSONPathExtractorGui" testclass="JSONPathExtractor" testname="Extrair ${VAR_NAME}" enabled="true">
  <stringProp name="JSONPostProcessor.referenceNames">${VAR_NAME}</stringProp>
  <stringProp name="JSONPostProcessor.jsonPathExprs">${JSON_PATH}</stringProp>
  <stringProp name="JSONPostProcessor.match_numbers">0</stringProp>
  <stringProp name="JSONPostProcessor.defaultValues">NOT_FOUND</stringProp>
</JSONPathExtractor>
<hashTree/>
```

---

## Referência de Substituição de Variáveis

| Placeholder | Descrição |
|---|---|
| `${SCENARIO_NAME}` | Nome do cenário de teste |
| `${CSV_HEADERS}` | Nomes de cabeçalho do CSV separados por vírgula |
| `${CONCURRENCY}` | Número de usuários virtuais |
| `${RAMP_UP_SECONDS}` | Duração do ramp-up em segundos |
| `${HOLD_SECONDS}` | Duração da sustentação em segundos |
| `${HTTP_SAMPLERS}` | Blocos de sampler HTTP renderizados |
| `${ENDPOINT_NAME}` | Nome legível do sampler |
| `${ENDPOINT_PATH}` | Caminho da URL (ex.: `/api/v1/products`) |
| `${REQUEST_BODY}` | Corpo JSON para requisições POST/PUT |
| `${VAR_NAME}` | Nome da variável para extração |
| `${JSON_PATH}` | Expressão JSONPath (ex.: `$.data.id`) |
| `${base_url}` | Em tempo de execução: carregado do `.properties` |
| `${auth_token}` | Em tempo de execução: carregado do `.properties` |

---

## Regras de Temporização

| Parâmetro | Pipeline (Modos 1 e 2) | Temporário (Modo 3) |
|---|---|---|
| Concorrência máxima | 50 | Definido pelo usuário |
| Ramp-up máximo | 2 minutos | Definido pelo usuário |
| Hold-for máximo | 10 minutos | Definido pelo usuário |
| Duração total máxima | 10 minutos | Definido pelo usuário |
