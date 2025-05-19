# Relatório da Sprint 2 - Verificação de Informações Sensíveis usando SAST

## **Introdução**
Dando continuidade ao trabalho iniciado na Sprint 1, este relatório detalha a análise focada na detecção de informações sensíveis no código da API **mec-energia-api**. Utilizamos os resultados da ferramenta **Bandit** para identificar a presença de informações como APIs, usuários e senhas hardcoded, além de outros dados sensíveis que possam comprometer a segurança do projeto.

## **Análise de Informações Sensíveis**

O Bandit fornece verificações específicas para detectar padrões que indicam a presença de informações sensíveis. Durante esta sprint, priorizamos a inspeção de códigos relacionados às seguintes vulnerabilidades:

### **Padrões Verificados**
- **Chaves de API (B105)**: Detecta uso de chaves de API expostas em arquivos específicos.
- **Senhas hardcoded (B106)**: Verifica senhas armazenadas diretamente no código.
- **Credenciais de usuários (B107)**: Identifica strings que podem conter credenciais expostas.
- **Outros dados sensíveis**: Strings específicas que indicam informações sigilosas.

### **Execução e Configuração**

#### Configuração da Pipeline
O mesmo pipeline configurado na Sprint 1 foi reutilizado. Alterações foram feitas para priorizar a detecção de informações sensíveis.

```yaml
sast:
  stage: test
  image: python:3.11
  before_script:
    - pip install bandit
  script:
    - bandit -r . -f json -o sast-report.json
    - bandit -r . -f json -o sast-password-report.json -c .bandit.yml
  artifacts:
    reports:
      sast: sast-report.json
      sast-password: sast-password-report.json
    expire_in: 1 week
```

#### Comando Local para Verificação de Informações Sensíveis

Para rodar localmente, o comando abaixo foi utilizado:

```bash
bandit -r . -f json -o sast-password-report.json -c .bandit.yml
```

O arquivo `.bandit.yml` foi configurado para priorizar verificações relacionadas a informações sensíveis, removendo tests e scripts dos arquivos testados ja que não são necessários para a análise de informações sensíveis.

```yaml
# Configuração do .bandit.yml para análise de informações sensíveis
tests:
  - B105
  - B106
  - B107

skips: []

exclude_dirs: ["./scripts/*", "./tests/*"]
```


## **Resultados Obtidos**

### **Relatório de Vulnerabilidades**
A análise identificou os seguintes problemas de segurança:

- **1 Senha hardcoded** detectada, que comprometeria a assinatura criptográfica da aplicação.
- **0 Chaves de API expostas** identificadas.
- **0 Credenciais de usuário expostas** encontradas.

#### Resumo dos Achados:
| Tipo de Informação Sensível | Ocorrências | Arquivo(s) Afetado(s) |
|-----------------------------|-------------|-----------------------|
| Senhas hardcoded           | 1           | `/mec_energia/settings/development.py`          |
| Chaves de API              | 0           |  |
| Credenciais de Usuário     | 0           |           |


### **Senha hardcoded encontrada**

**Arquivo:** `./mec_energia/settings/development.py`
  - **Linha:** 11
  - **Código:** `SECRET_KEY = "django-insecure-#123!@#$%^&*()_+"`
  - **Severidade:** Altíssima
  - **Descrição:** A chave `SECRET_KEY` é usada para fornecer assinatura criptográfica e deve ser definido com um valor exclusivo e imprevisível. Está hardcoded no arquivo de configuração `development.py`, o que pode comprometer a segurança da aplicação. É gerado usando um processo bem conhecido a partir de uma fonte de entropia cuja qualidade e segurança não podem ser garantidas (pelo Django).
  - **Mais Informações:** [Documentação de segurança do Django](https://docs.djangoproject.com/en/5.1/ref/settings/#secret-key), [Documentação do Bandit B105](https://bandit.readthedocs.io/en/1.8.0/plugins/b105_hardcoded_password_string.html)

## **Análise Comparativa com o OWASP Top 10**

A presença de informações sensíveis hardcoded no código, como senhas, chaves de API e outros segredos, está diretamente alinhada com os riscos destacados pelo OWASP Top 10, em especial os seguintes itens:

### A01:2021 – Broken Access Control
- **Descrição no OWASP**: Falhas em controles de acesso permitem que usuários não autorizados acessem funcionalidades ou dados restritos.
- **Relevância**: Expor a `SECRET_KEY` pode comprometer a integridade das assinaturas criptográficas, possibilitando a criação de tokens falsos, o que pode violar restrições de acesso e expor dados sensíveis.

### A02:2021 – Cryptographic Failures
- **Descrição no OWASP**: Dados sensíveis expostos devido ao uso incorreto ou fraco de algoritmos e práticas criptográficas.
- **Relevância**: Uma `SECRET_KEY` hardcoded é um exemplo claro de má prática em segurança criptográfica. Caso a chave seja comprometida, todas as funcionalidades que dependem dela, como sessões, tokens e assinaturas, ficam vulneráveis.

### A04:2021 – Insecure Design
- **Descrição no OWASP**: Falta de práticas de segurança integradas ao design do sistema.
- **Relevância**: Armazenar senhas ou chaves diretamente no código é um exemplo de um design inseguro. Um design seguro prevê o uso de práticas como variáveis de ambiente ou ferramentas de gerenciamento de segredos.

### A05:2021 – Security Misconfiguration
- **Descrição no OWASP**: Configurações de segurança incorretas ou padrão podem expor o sistema a ataques.
- **Relevância**: Hardcoding de informações sensíveis pode ser visto como uma configuração insegura que facilita a exploração de vulnerabilidades por atacantes.

## **Ações de Correção para a Senha Hardcoded**

1. **Remover a `SECRET_KEY` do Código Fonte:**
   - Utilize variáveis de ambiente para definir a `SECRET_KEY`.
   - Exemplo em Python:
     ```python
     import os
     SECRET_KEY = os.getenv('SECRET_KEY', 'default-fallback-key')
     ```

2. **Configurar uma Ferramenta de Gerenciamento de Segredos:**
   - Ferramentas como AWS Secrets Manager, Azure Key Vault ou HashiCorp Vault podem ser usadas para armazenar e acessar valores sensíveis de forma segura.

3. **Adicionar a `SECRET_KEY` ao Ambiente:**
   - No ambiente de desenvolvimento, defina a variável de ambiente:
     ```bash
     export SECRET_KEY="django-insecure-#123!@#$%^&*()_+"
     ```
   - Em servidores de produção, utilize configurações específicas do provedor de hospedagem para definir variáveis de ambiente.

4. **Adicionar o Arquivo de Configuração ao `.gitignore`:**
   - Caso use um arquivo local para armazenar segredos durante o desenvolvimento, garanta que ele não seja versionado:
     ```bash
     echo ".env" >> .gitignore
     ```

5. **Rotacionar Chaves Comprometidas:**
   - Gere uma nova `SECRET_KEY` para substituir a chave comprometida.
   - Exemplo de geração no Django:
     ```bash
     python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
     ```
   - Atualize todas as instâncias da aplicação com a nova chave.

6. **Monitoramento Contínuo:**
   - Integre ferramentas de SAST como o Bandit no pipeline para evitar novos hardcoded de segredos.


## **Recomendações Futuras**

- **Monitoramento contínuo**: Implementar ferramentas complementares como o GitLeaks para verificar informações sensíveis em históricos de commits.
- **Treinamento da equipe**: Promover workshops sobre boas práticas no uso de informações sensíveis.
- **Políticas de revisão**: Atualizar os critérios de revisão de código para incluir inspeção de informações sensíveis.


A detecção e remoção de informações sensíveis fortalecem a segurança do projeto e previnem riscos relacionados à exposição acidental de dados. Na próxima sprint, o foco será automatizar ainda mais os processos de segurança e expandir a cobertura de testes.
