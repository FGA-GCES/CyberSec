# Relatório de Implementação de SAST - Projeto EJ

## Introdução

Este relatório documenta a implementação de Static Application Security Testing (SAST) na pipeline de CI/CD do projeto EJ-Application. A ferramenta escolhida foi o Bandit, específica para análise de segurança em código Python, conforme requisitos do projeto.

## Objetivos

- Implementar uma ferramenta SAST (Bandit) na pipeline de CI/CD
- Configurar a análise para identificar vulnerabilidades de segurança no código
- Integrar a ferramenta de forma não-bloqueante na fase inicial
- Documentar vulnerabilidades encontradas e próximos passos

## Metodologia

### 1. Análise do Repositório

Inicialmente, foi realizada uma análise do repositório [ej-application](https://gitlab.com/pencillabs/ej/ej-application) para identificar:
- Tecnologias utilizadas (exclusivamente Python)
- Estrutura do projeto e organização do código
- Configuração atual da pipeline (.gitlab-ci.yml)
- Ferramentas de qualidade já implementadas

### 2. Configuração do Bandit

Foi criado um arquivo de configuração `.bandit.yaml` na raiz do projeto com as seguintes definições:

```yaml
exclude_dirs:
  - /tests
  - /venv
  - /migrations
  - /.git
  - /.tox
  - /lib

skips:
  - B101  # assert_used
  - B311  # random
```

Esta configuração:
- Exclui diretórios não relevantes para análise de segurança
- Ignora alguns testes específicos para reduzir falsos positivos

### 3. Integração na Pipeline

Foi adicionado um novo estágio "security" na pipeline do GitLab, com um job dedicado para o Bandit:

```yaml
bandit:
    stage: security
    image:
        name: python:3.9.19-slim-bookworm
        pull_policy: always
    before_script:
        - /bin/bash -c "pip install bandit==1.8.3"
    script:
        - /bin/bash -c "bandit -r src -c .bandit.yaml -f html -o bandit-report.html"
        - /bin/bash -c "bandit -r src -c .bandit.yaml -ll -ii"
    artifacts:
        paths:
            - bandit-report.html
        expire_in: 1 week
        when: always
    allow_failure: true
```

A configuração inclui:
- Uso da mesma versão do Python utilizada no projeto
- Geração de relatório HTML para análise detalhada
- Configuração `allow_failure: true` para não bloquear a pipeline na fase inicial

## Resultados

### Pipeline Executada

A implementação foi testada com sucesso na pipeline do GitLab:

![Pipeline SAST](https://github.com/user-attachments/assets/f80460f2-9673-4d93-ba34-fafb969ac69b)

![Image](https://github.com/user-attachments/assets/1adc1894-85fa-4a1e-b4dc-e309153b60b3)

A pipeline completa agora inclui:
- Verificação de estilo (black)
- Análise de lint (ruff)
- Análise de segurança (bandit)
- Testes unitários (pytest)

### Vulnerabilidades Encontradas

O Bandit identificou várias vulnerabilidades no código, incluindo:

![Vulnerabilidades Bandit](https://private-user-images.githubusercontent.com/62817427/446682232-985e7027-b705-4016-961e-b93626865951.png?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDc5MzQxNjYsIm5iZiI6MTc0NzkzMzg2NiwicGF0aCI6Ii82MjgxNzQyNy80NDY2ODIyMzItOTg1ZTcwMjctYjcwNS00MDE2LTk2MWUtYjkzNjI2ODY1OTUxLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA1MjIlMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNTIyVDE3MTEwNlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWJlNTAzMjg4ZmQ0MjExMWUwZDZhYmFhNzczOTFlOGVmMDU4ZWRmMGUyYjc3YmU2ZGU3YzdkOTVhNGRkMGY0ZDcmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.oDO7mZYbbbIEnTRuWd-RxfbszIXZu4kfFEpDzu6MlIE)

#### Principais vulnerabilidades:

1. **Uso de exec() (Risco Médio)**
   - **Arquivo**: src/ej/settings/themes.py, linha 23
   - **Descrição**: Uso da função `exec()` que pode permitir execução de código arbitrário
   - **CWE**: CWE-78

2. **Jinja2 sem autoescape (Risco Alto)**
   - **Arquivo**: src/ej_integrations/mailing.py, linha 60
   - **Descrição**: Ambiente Jinja2 configurado sem autoescape, podendo levar a vulnerabilidades XSS
   - **CWE**: CWE-94

3. **Uso de hash MD5 fraco (Risco Alto)**
   - **Arquivo**: src/ej_profiles/models.py, linha 326
   - **Descrição**: Uso de MD5 para hashing, considerado criptograficamente fraco
   - **CWE**: CWE-327

4. **Try-Except-Pass (Risco Baixo)**
   - **Múltiplos arquivos**
   - **Descrição**: Blocos try-except vazios que podem ocultar erros importantes
   - **CWE**: CWE-703

5. **Chamadas a requests sem timeout (Risco Médio)**
   - **Arquivo**: src/ej_integrations/utils.py, linha 6
   - **Descrição**: Chamadas HTTP sem timeout definido, podendo levar a ataques DoS
   - **CWE**: CWE-400

### Métricas

```
Metrics:
Total lines of code: 13845
Total lines skipped (#nosec): 0

Run metrics:
    Total issues (by severity):
        Undefined: 0
        Low: 7
        Medium: 2
        High: 2
    Total issues (by confidence):
        Undefined: 0
        Low: 1
        Medium: 0
        High: 10
```

## Recomendações

### Correções Prioritárias

1. **Alta Prioridade**:
   - Implementar autoescape no ambiente Jinja2 para prevenir XSS
   - Substituir o uso de MD5 por algoritmos mais seguros como SHA-256 ou bcrypt

2. **Média Prioridade**:
   - Revisar e refatorar o uso de `exec()` para alternativas mais seguras
   - Adicionar timeouts em todas as chamadas HTTP

3. **Baixa Prioridade**:
   - Revisar blocos try-except vazios para incluir tratamento adequado de erros

### Próximos Passos

 - Aplicar os conhecimentos adquiridos em DevSecOps para identificar possíveis vulnerabilidades no ciclo de desenvolvimento e propor melhorias.

 - Trabalhar na resolução de issues abertas, priorizando aquelas relacionadas à segurança, estabilidade e boas práticas de desenvolvimento seguro.

## Conclusão

A implementação do Bandit como ferramenta SAST na pipeline do projeto EJ-Application foi concluída com sucesso. A análise identificou vulnerabilidades importantes que devem ser tratadas para melhorar a segurança da aplicação.

A configuração atual permite que a equipe se adapte gradualmente às análises de segurança, sem bloquear o desenvolvimento, enquanto trabalha na correção das vulnerabilidades existentes.

Esta implementação representa um primeiro passo importante na direção de práticas de desenvolvimento seguro, que podem ser expandidas e aprimoradas no futuro.
