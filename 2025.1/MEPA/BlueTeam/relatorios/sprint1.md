# SPRINT 1 - MEPA BLUE TEAM

## Integrantes

    - Ricardo Augusto Valle Maciel - 180077899 - @avmricardo
    - Mateus Orlando Medeiros Ribeiro - 211062259 - @MateusPy
    - Danilo Carvalho Antunes

## 🛡️ Atividades de Capacitação Realizadas

1 - ✅ Conclusão da trilha “Pre Security” – TryHackMe

Conteúdos abordados:

 - Fundamentos de segurança defensiva (Blue Team).

 - Operação de Security Operations Center (SOC).

 - Threat Intelligence (Inteligência de Ameaças).

 - Digital Forensics and Incident Response (DFIR).

 - Malware Analysis (Análise de Malware).


2 - ✅ Conclusão da trilha “DevSecOps” – TryHackMe

Conteúdos abordados:

 - Integração da segurança no ciclo de desenvolvimento (DevSecOps).

 - Práticas de segurança em pipelines CI/CD.

 - Ferramentas e técnicas para automação de análise de vulnerabilidades no desenvolvimento.

 - Gerenciamento de dependências e análise de containers.


## 💻 Atividades Práticas Realizadas

1 - ✅ Ambiente do Projeto MEPA configurado localmente:

 - Clonamos o repositório do projeto.

 - Realizamos o setup do ambiente na nossa máquina (configuração de dependências, banco de dados e serviços necessários).

 - Iniciamos o processo de análise do código e levantamento de pontos de atenção relacionados à segurança e possíveis melhorias.

2 - ⚙️ Análise inicial das Issues:

 - Começamos a estudar a Issue aberta no repositório, a qual pedia a criação de um SAST(Static Application Security Testing) caso o projeto não possuísse um.

 - O que é um SAST: É uma técnica de análise de segurança que examina o código fonte, bytecode ou binário de um software sem executá-lo.

 - Analisamos o repositório [MEPA Energia API](https://gitlab.com/lappis-unb/projetos-energia/mec-energia/mec-energia-api) e observamos que não havia SAST implementado na pipeline do projeto. Então fomos estudar a ferramenta Bandit que foi indicado pelo monitor na própria issue.

 - Após um estudo detalhado da documentação do Bandit, [disponível aqui](https://bandit.readthedocs.io/en/latest/), o integrante Ricardo Augusto criou um fork para implementarmos um JOB de SAST na pipeline do projeto.

 - Ao final, o JOB que criamos ficou assim:

    ```
    sast-bandit:
        stage: quality
        image: python:3.11
        before_script:
            - pip install bandit
        script:
            - echo "Rodando Bandit (SAST) no projeto"
            - bandit -r . -ll -f json -o bandit-report.json
        artifacts:
            paths:
                - bandit-report.json
            when: always
            expire_in: 1 week
        allow_failure: true
        rules:
            - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
            - if: $CI_PIPELINE_SOURCE == 'push'
    ```

- Vamos detalhar melhor o que cada parte faz:

 - 1: Bloco Principal - Define o nome do job. Esse nome aparece na interface do GitLab CI/CD quando o pipeline roda.

    ```
    sast-bandit:
    ```

 - 2: Stage - O job será executado na etapa chamada quality.
Essa etapa geralmente agrupa jobs relacionados a qualidade de código, análise estática (SAST), linters, etc.

    ```
    stage: quality
    ```

 - 3: Imagem - O job usará uma imagem Docker com Python 3.11 como ambiente de execução.

    ```
    image: python:3.11
    ```


 - 4:  Antes de Rodar (before_script) - Antes de executar o script principal, instala o Bandit, que é a ferramenta de análise estática de segurança para código Python.

    ```
    before_script:
        - pip install bandit
    ```


 - 5: Script Principal
    
    - Imprime a mensagem "Rodando Bandit (SAST) no projeto" no log do pipeline, de forma informativa.

    - Comando que roda o Bandit(bandit -r . -ll -f json -o bandit-report.json):

        - -r . → Faz uma análise recursiva na raiz (.) do projeto.

        - -ll → Define o nível de detalhe do log como "low severity" (baixo), mostrando mais informações.

        - -f json → Formata o output no formato JSON.

        - -o bandit-report.json → Salva o resultado no arquivo chamado bandit-report.json.

    ```
    script:
        - echo "Rodando Bandit (SAST) no projeto"
        - bandit -r . -ll -f json -o bandit-report.json
    ```

 - 6: Artefatos

    - paths → Indica quais arquivos serão salvos como artefatos do job (bandit-report.json).

    - when: always → Salva os artefatos sempre, mesmo se o job falhar.

    - expire_in: 1 week → Os artefatos ficam disponíveis para download no GitLab por 1 semana.

    ```
    artifacts:
        paths:
            - bandit-report.json
        when: always
        expire_in: 1 week
    ```
 - 7: Permitir Falha (não quebra o pipeline) - Mesmo que o Bandit encontre vulnerabilidades (ou até erros), o pipeline não falha, ele apenas registra o problema. Isso permite que o pipeline continue executando as próximas etapas.

    ```
    allow_failure: true
    ```

 - 8: Regras de Execução - Define quando esse job será executado. 
    
    - Se o pipeline for disparado por um Merge Request (merge_request_event), o job roda.

    - Se o pipeline for disparado por um Push (push) no repositório, o job também roda.

    ```
    rules:
        - if: $CI_PIPELINE_SOURCE == 'merge_request_event'
        - if: $CI_PIPELINE_SOURCE == 'push'
    ```

## Resultados

    - A pipeline rodou conforme o esperado, abaixo disponibilizamos alguns prints dos testes realizados pelos alunos Mateus Orlando e Ricardo Augusto:

        - 1: Imagem de um merge teste antes de subirmos o JOB do Bandit:

            (imagema aqui)

        - 2: Imagem do merge após a integração do JOB do Bandit:

            (imagem aqui)

        - 3: Imagens do arquivo de relatório .jason, que foi criado após a passagem na pipeline.
            - Imagem 1:
                (imagem)
            - Imagem 2:
                (imagem)
        
        - 4: Imagem mostrando que a pipeline gerou Warnings. Os quais são motivamos por pontos de possíveis SQL injections.

            (imagem aqui)
    
    - Então, como resultado foram apontados 0 problemas de severidade alta, 6 problemas de severidade média, e 356 de baixa.

        Métricas Principais
            Total de arquivos analisados: 175
            Problemas encontrados: 6
            Problemas por gravidade:
                Severidade Média: 6
                Severidade Baixa: 356
            Problemas por confiança:
                Alta confiança: 321
                Média confiança: 38
                Baixa confiança: 3



## 📈 Próximos Passos

 - Aprofundar o entendimento da arquitetura do projeto MEPA.

 - Aplicar os conhecimentos adquiridos em DevSecOps para identificar possíveis vulnerabilidades no ciclo de desenvolvimento e propor melhorias.

 - Trabalhar na resolução de issues abertas, priorizando aquelas relacionadas à segurança, estabilidade e boas práticas de desenvolvimento seguro.
