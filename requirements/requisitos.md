Eu preciso criar com base nas informações a seguir um prompt para criação de uma habilidade de agente chamada **test-performance**. Esta habilidade será usada para executar inicialmente no copilot e no devin, mas ela deve ser capaz de ser utilizada em qualquer agente, seja ele Codex, Claude Code ou Gemini. 

Uma skill de agente possui a seguinte estrutura:
```
spec-driven-development/
├── SKILL.md
├── references/ # templates ou knowledge reusable
└── scripts/ # procedimentos executáveis pelo agente
```



# Modadlidades da skill

Essa habilidade terá estas funções:
- Gerar e manter testes de performance a partir da codebase da aplicação para execução via pipeline:
  - Nesta modalidade como é um testes que será realizado antes de implantar a aplicação a ideia é fazer um teste sustendado por um tempo máximo de 10 min conforme configuração p4all.yml. A estratégias de testes adotada sempre será um load test com uma rampa de subida e com uma sustentação por um tempo e finalização do teste, conforme é possível configurar através do p4all.yml.
  - Deve ser analisado o as APIs que são expostas pela aplicação que estão no diretório **/app** para compor uma ordem de sequenciamento e de pendência entre as APIs e conseguir determinar a estrutura e gerar a massa de testes, bem como gerar arquivos de grafo de sequenciamento e dependências e também arquivos de configuração de variáveis de ambiente do teste. 
  - Aqui deve ser gerado na pastas **tests/performance/<env>/** todos os arquivos esperados da estrtura. E precias ser feito para os dois ambientes **dev** e **hom**
  - O arquivo de variaviesd e ambiente pode ser gerado com placeholders para o usário editar depois e configurar corretamente as variáveis. Porcure deixar comenentários para que ele possa entender cada parâmetro
  - O arquivo p4all.yml deve ser configurado com sua assistência garantindo o consistência da configuração
  - O cenário para o contexto da pipeline, assim como os nomes dos arquivos deve ser 'default', uma vez que não vem exatamente variação de cenário.

- Gerar e manter testes de performance a partir dos contratos de api gateway para execução via pipeline:
  - Nesta modalidade como é um testes que será realizado antes de implantar a aplicação a ideia é fazer um teste sustendado por um tempo máximo de 10 min conforme configuração p4all.yml. A estratégias de testes adotada sempre será um load test com uma rampa de subida e com uma sustentação por um tempo e finalização do teste, conforme é possível configurar através do p4all.yml.
  - Deve ser analisado os contratos dos API GATEWAYS que estão no diretório **/contratos** para compor uma ordem de sequenciamento e de pendência entre as APIs e conseguir determinar a estrutura e gerar a massa de testes, bem como gerar arquivos de grafo de sequenciamento e dependências e também arquivos de configuração de variáveis de ambiente do teste.  
  - Aqui deve ser gerado na pastas **tests/performance/<env>/** todos os arquivos esperados da estrtura. E precias ser feito para os dois ambientes **dev** e **hom**
  - O arquivo de variaviesd e ambiente pode ser gerado com placeholders para o usário editar depois e configurar corretamente as variáveis. Porcure deixar comenentários para que ele possa entender cada parâmetro
  - O arquivo p4all.yml deve ser configurado com sua assistência garantindo o consistência da configuração
  - O cenário para o contexto da pipeline, assim como os nomes dos arquivos deve ser 'default', uma vez que não vem exatamente variação de cenário.


- Gerar e manter testes de perofmrance a partir dos contratos de api gateway para execução em ambiente temporário:
  - Deve ser analisado os contratos dos API GATEWAYS que estão no diretório **/contratos** para compor uma ordem de sequenciamento e de pendência entre as APIs e conseguir determinar a estrutura e gerar a massa de testes, bem como gerar arquivos de grafo de sequenciamento e dependências e também arquivos de configuração de variáveis de ambiente do teste. 
  - O usuario deverá ter a possibildiade gerar cenários de testes usando as estratégias de load, stress, endurance/ soak
  - O Agente deve guialo na definição da curva de testes, questionando:
    - Configurações de Fases (Rampu-up, Sustentação, Ramp-Down)
    - Cargas nominais
    - Caras de pico
    - Durações por fase
    - Carga (Usuários virutais)

  - Aqui deve ser gerado na pastas **tests/performance/<env>/** todos os arquivos esperados da estrtura. E precias ser feito para os dois ambientes **dev** e **hom**. O único arquivo que não deve ser gerado é o p4all.yml, uma vez que nesta função não será uma execução via pipeline, mas sim via um nó temporário (EC2);
  - O arquivo de variavies de ambiente pode ser gerado com placeholders para o usário editar depois e configurar corretamente as variáveis. Porcure deixar comenentários para que ele possa entender cada parâmetro
  - O cenario deve ser gerado com o nome do cenário que deve ser questiondo pelo usuário, concatenado com a estratégia de testes selecionado pelo usário
    - nome para os arquivos: [nome_cenário]_[load | stress | endurance]
  - O agente deve guiar o usuário nas definições.



# Estrutura básica do repositório de projeto que possui aplicação
```
/
├── app/ # Onde se encontra estrutura da aplicação e pode ser diferentes tipos de implementações (SpringBoot/ Kotlin/ Java, Python, Golang e etc)
├── infra/ # Onde se encontra scripts para implantação de infraestrutura como manifestos do Kubernetes ou scripts terraform
└── tests/ # Onde sen ocntram configurações de teste de performance e de outros tipos de teste que são executados em etapas exclusivas da pipeline. Testes que são diferentes dos normalmente implementados com a aplicação
```

# Estrutura básica do repositório de projeto que possui scripts de perfomrance gerados em aplicações como referência
```
/
├── contratos/
│   ├── <openapi_gateway_1>.yaml
│   ├── <openapi_gateway_2>.yaml
│   ├── <openapi_gateway_3>.yaml
│   └── <openapi_gateway_n>.yaml
└── tests/ # Onde será montando o testes de performance integrado
```

# A habilidade opera nesta estrutura de diretório dentro do meu projeto:

```
tests/
└── performance
    ├── dev
    │   ├── p4all.yml # Arquivo de configuração para execução dos testes via pipeline
    │   ├── <scenario_name>.jmx # script de teste do jmeter
    │   └── <scenario_name>.csv # Massa para uso no script de testes
    │   └── <scenario_name>.properties # Arquivo com varáivies usadas no script de teste
    │   └── <scenario_name>_graph.yml # grafo sequência de execução das API mapeadas para o scenario considerando as suas dependências
    └── hom # mesma estrutura de dev mas com configurações para hom
```

# Estrutura do arquivo de configuração p4all.yml
```
execution:
  concurrency: 10 # Máximo de 50 usuários virtuais
  ramp-up: 5s # Máximo de 2 minutos (2m)
  hold-for: 30s #Máixmo de 10 minutos (10m)
  scenario: <scneario_name> # Nome do cenário que é configurados mais a frente
scenarios:
  <scneario_name>:
    script: <scneario_name>.jmx
p4all:
  limit:
    rps: 300 # Mínimo de requisições por segundo
    error-percent: 4 # Máixmo de erros (porcentagem)
    time:
      p90: 500 # Máximo de percentil 90 (Milissegundos)
      avg: 500 # Máixmo de tempo médio (Milissegundos)
```