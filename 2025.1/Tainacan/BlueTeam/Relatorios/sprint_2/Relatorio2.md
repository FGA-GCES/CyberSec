
# Relatório de segurança sobre a exposição de credenciais sensíveis (Sprint 2)

## Objetivo

Este relatório apresenta uma análise do projeto Tainacan quanto a senhas, informações sensíveis ou quaisquer dados semelhantes que possam comprometer a segurança do sistema. O documento abrange a análise manual e descobertas recentes obtidas com o uso da ferramenta **Gitleaks**, que escaneia todo o histórico do repositório em busca de segredos.

## 1. Análise dos arquivos referêntes ao workflow

### 1.1 Análise do arquivo `/workflows/test.yml`
  
- **Sem exposição de chaves de API ou tokens**.
- **Sem uso de senhas hardcoded diretamente no arquivo**.

### 1.2 Análise do script `install-wp-tests.sh`

- **As credenciais (DB_NAME, DB_USER, DB_PASS)** são recebidas via argumentos, e não hardcoded.

### 1.3 Riscos e Oportunidades de Melhoria

```plaintext
- Descrição: A senha do banco é passada como argumento do script ($3) e, posteriormente, usada diretamente no comando
  mysqladmin --password=..., o que a torna visível em ferramentas como ps, no histórico de shell ou em logs de CI/CD.
- Risco: Exposição de senha por argumentos de linha de comando e comandos internos
- Recomendação: Evitar o uso de senhas como argumento ou diretamente nos comandos. 
  Preferir o uso de variáveis de ambiente seguras. Armazenar a senha em um arquivo de configuração fora do
  versionamento (como .env).
```

## 2. Uso do Gitleaks para Detecção de Segredos

### 2.1 O que é o Gitleaks?

- Ferramenta open-source para detectar **segredos, tokens, chaves de API, senhas, e outras informações sensíveis** em repositórios Git.
- Pode escanear tanto os arquivos atuais quanto o **histórico completo de commits**, encontrando segredos mesmo que tenham sido removidos posteriormente.

### 2.2 Como o Gitleaks funciona?

- **Escaneia todo o histórico Git por padrão**: percorre cada commit, analisando cada arquivo naquela versão.
- Isso significa que segredos expostos em commits antigos **serão reportados mesmo que não existam mais no código atual**.
- Também pode rodar em modo “snapshot” para analisar apenas o estado atual dos arquivos.

### 2.3 Resultados do Scan no Repositório

- Foram encontrados **4 leaks** no histórico, incluindo:
  - Chaves genéricas de API hardcoded.
  ```plaintext
  - File: src/classes/importer/class-tainacan-flickr-importer.php
  - Linha: 13
  - Leak: apiKeyValue = 'xxxxxxxxxxx'
  - Commit: Ainda presente no repositório atual e encontrado nos commits: 47ea0d6a16f62cc369d0db9fc7853add2652a4c7 e 5415ef525b2d28d77efd864f2f496061785e78a7
  - Descrição: Essa chave é usada para acessar os serviços da API pública do Flickr. 
    Apesar de APIs públicas como essa parecerem inofensivas à primeira vista, manter a chave embutida diretamente no
    código-fonte público pode trazer problemas, especialmente se a conta estiver associada a funcionalidades 
    avançadas ou integradas com credenciais adicionais.
  - Riscos: Uso indevido por terceiros, que podem consumir a cota da API, resultando em indisponibilidade para os usuários legítimos. 
    Associação reputacional, com qualquer uso malicioso da chave (como para scraping ou spam) podendo recair sobre o titular da conta Flickr, 
    impactando negativamente a reputação do projeto.
  ```
  - Chave de API do Google Cloud Platform (GCP).
  ```plaintext
  - File: src/classes/importer/class-tainacan-youtube-importer.php
  - Linha: 27
  - Leak: 'xxxxxxxxxxxx'
  - Commit: c1603aa9c1c2663e75e2ea5640485248b9df15c0
  - Descrição: Essa chave pertence aos serviços da Google Cloud, provavelmente usada para a API do YouTube. 
    Embora muitas dessas chaves tenham restrições de domínio ou IP, ao estarem publicadas, ficam sujeitas a engenharia reversa, 
    scraping automatizado e abusos por terceiros.
  - Riscos: Consumo indevido de cotas da API, gerando custos para o projeto. Possível uso para coleta de dados de forma automatizada e indevida 
    (ex: vídeos, estatísticas). Se não estiver limitada por IP ou domínio, pode ser usada para ataques de negação de serviço (DoS) ou atividades ilícitas.
  ```
  - Token em script shell.
  ```plaintext
  - File: tainacam/run-cypress.sh
  - Linha: 16
  - Leak: cy_record_key='xxxxxxxxxxxxx'
  - Commit: a4bf702203087dcc99455101df9a67ab6507e3ab
  - Descrição: Trata-se de uma chave genérica que parece ser usada em testes automatizados com Cypress (cy_record_key). 
    Mesmo se usada apenas em ambientes de desenvolvimento, essa chave também deve ser mantida fora do repositório.
  - Riscos: Possível acesso por terceiros aos resultados de testes, logs ou falhas internas. 
    Exposição de pipelines de testes ou comportamento da aplicação em execução.
  ```

### 2.4 O que o relatório do Gitleaks mostra?

- O relatório **exibe o trecho exato encontrado, o arquivo, linha, commit e autor do commit**.
- Mostra detalhes como o tipo de segredo detectado, para facilitar a análise.
- Não é apenas um indicativo de arquivo suspeito, ele mostra exatamente o conteúdo que foi considerado leak.

## 3. Conclusão Geral

- O histórico Git contém registros de segredos sensíveis que devem ser tratados para evitar exposição, além de senhas que ainda estão presentes no repositório atual.
- Passar credenciais via argumentos no script pode representar riscos adicionais.
