# Relatório de Análise de Secrets - Projeto EJ - Sprint 2

**Semestre:** 2025.1
**Sprint:** 2
**Grupo:** BlueTeam (Número do grupo a ser inserido)

## Introdução

Este relatório detalha a análise realizada no projeto EJ-Application e sua pipeline de CI/CD com o objetivo de identificar informações sensíveis (secrets) expostas, como chaves de API, tokens, senhas e outras credenciais hardcoded. A exposição de secrets representa um risco significativo de segurança. Nesta sprint, além da análise manual, foi integrada uma ferramenta automatizada de secret scanning (Gitleaks) à pipeline para detecção contínua.

## Objetivos

- Verificar a presença de informações sensíveis hardcoded no código-fonte e configurações.
- Implementar e integrar uma ferramenta automatizada de secret scanning (Gitleaks) na pipeline.
- Analisar arquivos de configuração e scripts de pipeline em busca de secrets expostos.
- Identificar riscos associados à exposição de secrets.
- Documentar os achados (manuais e automatizados) e fornecer recomendações para mitigação.

## Metodologia

A análise foi realizada utilizando as seguintes técnicas:

1.  **Análise Estática Manual (grep)**: Buscas iniciais por palavras-chave comuns associadas a secrets no código-fonte e arquivos de configuração.
2.  **Revisão de Configuração da Pipeline**: Análise do arquivo `.gitlab-ci.yml` para identificar o uso de tokens e variáveis sensíveis.
3.  **Revisão de Arquivos de Ambiente**: Análise do arquivo `docker/variables.env`.
4.  **Análise Automatizada (Gitleaks)**: Integração da ferramenta Gitleaks na pipeline de CI/CD para varredura automatizada do repositório em busca de segredos expostos, utilizando análise de padrões e entropia.

## Resultados da Análise

A análise combinada (manual e automatizada) identificou os seguintes pontos de atenção:

### 1. Secrets no Código-Fonte (`src/**/*.py`)

- **Análise Manual**: Identificou usos de termos como `password`, `token`, `secret_id`, majoritariamente relacionados a funcionalidades esperadas (gerenciamento de senhas, tokens de reset, etc.).
- **Análise Automatizada (Gitleaks)**: Não reportou segredos críticos diretamente no código da aplicação principal, mas identificou uma chave genérica em um arquivo de teste (`src/ej_users/tests/test_api.py`).
- **Risco**: Baixo para o código principal, Médio para a chave no arquivo de teste (se for uma chave real ou representativa de um padrão inseguro).
- **Recomendação**: Revisar a chave encontrada no arquivo de teste. Considerar o uso de placeholders ou dados fictícios em testes.

### 2. Secrets na Configuração da Pipeline (`.gitlab-ci.yml`)

- **`GITLAB_EJ_TOKEN: ...` (Linha 10)**
    - **Detectado por**: Análise Manual e Gitleaks (como `generic-api-key`).
    - **Descrição**: Token de deploy do GitLab hardcoded.
    - **Risco**: **Alto**.
    - **Recomendação**: Remover o token do arquivo e armazená-lo como uma variável de CI/CD protegida e mascarada no GitLab.

- **`https://gitlab-ci-token:${CI_JOB_TOKEN}@...` (Linha 82)**
    - **Detectado por**: Análise Manual.
    - **Descrição**: Uso seguro da variável `CI_JOB_TOKEN`.
    - **Risco**: Baixo.

- **`https://$GITLAB_INFRA_TOKEN@...` (Linha 114)**
    - **Detectado por**: Análise Manual.
    - **Descrição**: Uso da variável `GITLAB_INFRA_TOKEN`.
    - **Risco**: Médio (depende da configuração da variável).
    - **Recomendação**: Garantir que `GITLAB_INFRA_TOKEN` esteja definida como variável protegida e mascarada no GitLab.

### 3. Secrets em Arquivos de Ambiente (`docker/variables.env`)

- **`DJANGO_SECRET_KEY=...` (Linha 8)**
    - **Detectado por**: Análise Manual e Gitleaks (como `generic-api-key` ou similar, dependendo das regras do Gitleaks).
    - **Descrição**: Chave secreta do Django hardcoded.
    - **Risco**: **Alto**.
    - **Recomendação**: Remover do arquivo, carregar via variáveis de ambiente em produção/staging, adicionar `variables.env` ao `.gitignore`.

- **`JWT_SECRET=...` (Linha 23)**
    - **Detectado por**: Análise Manual e Gitleaks (como `generic-api-key` ou similar).
    - **Descrição**: Chave secreta JWT hardcoded.
    - **Risco**: **Alto**.
    - **Recomendação**: Seguir a mesma recomendação da `DJANGO_SECRET_KEY`.

- **`MAILGUN_API_KEY=` e `EMAIL_HOST_PASSWORD=`**
    - **Detectado por**: Análise Manual.
    - **Descrição**: Campos vazios para credenciais.
    - **Risco**: Baixo (enquanto vazios), mas risco potencial.
    - **Recomendação**: Garantir que `variables.env` esteja no `.gitignore`.

### 4. Outros Achados (Gitleaks)

- **JWT em Documentação (`docs/development-guides/pt-br/api.rst`)**: Gitleaks detectou um JWT hardcoded em um arquivo de documentação. Embora possa ser um exemplo, representa um risco se for um token válido ou se ensinar práticas inseguras.
    - **Risco**: Médio.
    - **Recomendação**: Substituir por um placeholder ou exemplo claramente inválido na documentação.

### 5. Integração do Gitleaks na Pipeline

Foi adicionado um job `gitleaks_scan` ao estágio `security` da pipeline:

```yaml
# Job para Secret Scanning com Gitleaks (com resumo em Markdown)
gitleaks_scan:
    stage: security
    image:
        name: zricethezav/gitleaks:latest
        entrypoint: ['']
    before_script:
        - apk add --no-cache jq bash
    script:
        - gitleaks detect --source . --verbose --report-path gitleaks-report.json || true
        - |
          if [ -s gitleaks-report.json ]; then
            echo '# Gitleaks Scan Summary' > gitleaks-summary.md
            # ... (script jq para gerar resumo) ...
          else
            echo '# Gitleaks Scan Summary' > gitleaks-summary.md
            echo 'No secrets found or report is empty.' >> gitleaks-summary.md
          fi
    artifacts:
        paths:
            - gitleaks-report.json
            - gitleaks-summary.md
        expire_in: 1 week
        when: always
    allow_failure: true
```

Este job executa o Gitleaks, gera um relatório JSON completo e um resumo em Markdown (`gitleaks-summary.md`) para facilitar a visualização dos resultados.

**Resumo dos Achados do Gitleaks (Exemplo do `gitleaks-summary.md` gerado):**

```markdown
# Gitleaks Scan Summary

## Secrets Found:

### Finding Type: generic-api-key
- **Description:** Detected a Generic API Key...
- **File:** .gitlab-ci.yml
- **Line:** 10
- **Secret:** `51xTbvyHGmEfeZZUxUct`
...
---
### Finding Type: jwt
- **Description:** Uncovered a JSON Web Token...
- **File:** docs/development-guides/pt-br/api.rst
- **Line:** 30
- **Secret:** `eyJhbGciOiJIUzI1NiIsIn...`
...
---
### Finding Type: generic-api-key
- **Description:** Detected a Generic API Key...
- **File:** src/ej_users/tests/test_api.py
- **Line:** 129
- **Secret:** `8d969eef6ecad3c29a3a6...`
...
---
```
*(Nota: O conteúdo completo do resumo gerado pela pipeline deve ser consultado no artefato `gitleaks-summary.md`)*

## Recomendações Gerais (Reforçadas pela Análise Automatizada)

1.  **Utilizar Variáveis de Ambiente / Secrets Management**: Prioridade máxima para remover `GITLAB_EJ_TOKEN`, `DJANGO_SECRET_KEY`, `JWT_SECRET` dos arquivos versionados.
2.  **`.gitignore`**: Adicionar `variables.env` imediatamente ao `.gitignore`.
3.  **GitLab CI/CD Variables**: Configurar os tokens e chaves necessários para a pipeline como variáveis protegidas e mascaradas.
4.  **Secret Scanning Contínuo**: Manter o Gitleaks ativo na pipeline. Considerar torná-lo bloqueante (`allow_failure: false`) para novos segredos detectados após a limpeza inicial.
5.  **Revisão de Código**: Manter a verificação de secrets como parte do processo de revisão.
6.  **Rotação de Secrets**: Rotacionar imediatamente o `GITLAB_EJ_TOKEN`, `DJANGO_SECRET_KEY` e `JWT_SECRET` expostos.
7.  **Limpeza de Histórico (Avançado/Opcional)**: Para segredos expostos há muito tempo, considerar ferramentas para limpar o histórico do Git (complexo e arriscado, avaliar necessidade).
8.  **Revisar Documentação e Testes**: Remover ou substituir secrets hardcoded em arquivos de documentação e testes.

## Conclusão

A análise combinada (manual e automatizada com Gitleaks) confirmou a presença de múltiplos secrets hardcoded, representando riscos significativos. A integração do Gitleaks na pipeline fornece um mecanismo de detecção contínua essencial. A implementação das recomendações, especialmente a remoção de secrets dos arquivos versionados e a rotação dos valores expostos, é crucial para melhorar a postura de segurança do projeto EJ-Application.
