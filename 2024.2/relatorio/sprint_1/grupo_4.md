# Relatório da Sprint 1 - SAST usando Bandit

## **Introdução**
Neste relatório, detalhamos a experiência ao configurar e utilizar o **Bandit** como ferramenta de **SAST (Static Application Security Testing)** para identificar vulnerabilidades na API **mec-energia-api**. O Bandit foi integrado ao pipeline `.gitlab-ci.yml` e também configurado para rodar manualmente, facilitando a identificação de problemas de segurança no código Python.

---

## **Como o Bandit funciona no SAST**

O Bandit analisa os arquivos Python no repositório para detectar padrões de código potencialmente inseguros. Ele verifica problemas como:
- Uso inseguro de funções de criptografia.
- Vazamento de informações sensíveis.
- Injeção de código.
- Uso incorreto de bibliotecas padrão.

A ferramenta foi configurada no pipeline para executar automaticamente em cada *merge request* e em *pushes* para branches específicas, gerando um relatório JSON com os resultados.

---

## **Execução na Pipeline**

### Configuração no `.gitlab-ci.yml`
Adicionamos o seguinte trecho ao pipeline para integrar o Bandit:

```yaml
sast:
  stage: test
  image: python:3.11
  before_script:
    - pip install bandit
  script:
    - bandit -r . -f json -o sast-report.json
  artifacts:
    reports:
      sast: sast-report.json
    expire_in: 1 week
```

### Como funciona:

- Ambiente: A imagem do Python 3.11 é utilizada.
- Instalação: O Bandit é instalado antes da execução.
- Análise: Ele analisa recursivamente (-r) o diretório do projeto e gera um relatório no formato JSON.
- Relatório: O resultado é anexado como artefato da pipeline e disponibilizado na aba "Segurança" do GitLab.

## **Execução manual**

Para rodar o Bandit manualmente, siga os passos abaixo:

### Pré-requisitos
Certifique-se de que o Python e o Bandit estão instalados no ambiente local:

```pip install bandit```

### Comando para análise

- Para gerar um relatório em JSON:

```bandit -r . -f json -o sast-report.json```

- Para ignorar determinados testes:

```bandit -r . -f json -o sast-report.json --skip B101,B102```

#### Teste do comando:  
![][running]

---

### Resultados e Benefícios

- Detectou vulnerabilidades iniciais: Foi possível identificar funções e bibliotecas que precisavam de ajustes para segurança.
- Relatório automatizado: A integração com o GitLab permite fácil visualização dos problemas de segurança.
- Execução simples: A ferramenta é leve e fácil de rodar, tanto na pipeline quanto localmente.

#### Alguns dos resultados gerados:  
![][results]

### Recomendações Futuras

- Expandir a análise adicionando outras ferramentas de SAST complementares, como o Semgrep.
- Automatizar a correção de vulnerabilidades recorrentes usando pré-commits ou hooks.

[running]: ../../Images/bandit_running.png
[results]: ../../Images/bandit_results.png