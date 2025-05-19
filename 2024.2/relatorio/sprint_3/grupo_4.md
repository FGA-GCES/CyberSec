# Relatório da Sprint 3 - Análise do Container Docker com Trivy

## **Introdução**
Neste relatório, detalhamos a experiência ao configurar e utilizar o **Trivy** como ferramenta de análise de segurança para detectar vulnerabilidades e informações sensíveis na imagem Docker utilizada pelo projeto **mec-energia-api**. O Trivy foi integrado ao pipeline `.gitlab-ci.yml` e também configurado para rodar manualmente, facilitando a identificação de problemas relacionados a dependências desatualizadas, configurações inseguras e segredos expostos.

---

## **Como o Trivy funciona**

O **Trivy Scanner** é uma ferramenta de segurança que realiza análises rápidas e precisas para identificar vulnerabilidades e informações sensíveis. Ele suporta diversas camadas e formatos de análise, permitindo sua aplicação em diferentes contextos de segurança.

### **Principais Funcionalidades**
1. **Análise de Vulnerabilidades em Dependências**: Detecta falhas de segurança em bibliotecas e pacotes utilizados no projeto.
2. **Verificação de Imagens Docker**: Inspeciona todas as camadas da imagem para identificar dependências vulneráveis e configurações inseguras.
3. **Detecção de Informações Sensíveis**: Localiza segredos como chaves de API, senhas hardcoded e outros dados confidenciais.
4. **Verificação de Infraestrutura como Código (IaC)**: Analisa arquivos como Terraform, Kubernetes e CloudFormation em busca de configurações inseguras.
5. **Integração com Pipelines CI/CD**: Automatiza verificações em pipelines de integração contínua.

### **Etapas de Funcionamento**
1. **Coleta de Dados**:
   - Extrai informações de pacotes, dependências e configurações da imagem Docker ou do código analisado.
2. **Consulta ao Banco de Dados de Vulnerabilidades**:
   - Verifica os pacotes analisados contra bancos de dados de vulnerabilidades como NVD, Alpine SecDB, Red Hat CVE Database, entre outros.
3. **Análise de Informações Sensíveis**:
   - Realiza uma varredura no código em busca de padrões que indiquem a presença de dados sigilosos.
4. **Geração de Relatórios**:
   - Produz relatórios detalhados (em formatos como JSON ou tabela), com informações sobre as vulnerabilidades encontradas, severidade e recomendações de correção.

---

## **Execução na Pipeline**

### Configuração no `.gitlab-ci.yml`
Adicionamos o seguinte trecho ao pipeline para integrar o trivy seguindo as recomendações da [documentação oficial](https://trivy.dev/latest/tutorials/integrations/gitlab-ci/):

```yaml
trivy:
  stage: test
  image: docker:stable
  services:
    - name: docker:dind
      entrypoint: ["env", "-u", "DOCKER_HOST"]
      command: ["dockerd-entrypoint.sh"]
  variables:
    DOCKER_HOST: tcp://docker:2375/
    DOCKER_DRIVER: overlay2
    # See https://github.com/docker-library/docker/pull/166
    DOCKER_TLS_CERTDIR: ""
    IMAGE: trivy-ci-test:$CI_COMMIT_SHA
  before_script:
    - export TRIVY_VERSION=$(wget -qO - "https://api.github.com/repos/aquasecurity/trivy/releases/latest" | grep '"tag_name":' | sed -E 's/.*"v([^"]+)".*/\1/')
    - echo $TRIVY_VERSION
    - wget --no-verbose https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.tar.gz -O - | tar -zxvf -
  allow_failure: true
  script:
    # Build image
    - docker build -t $IMAGE .
    # Build report
    - ./trivy --exit-code 0 --cache-dir .trivycache/ --no-progress --format template --template "@contrib/gitlab.tpl" -o gl-container-scanning-report.json $IMAGE
    # Print report
    - ./trivy --exit-code 0 --cache-dir .trivycache/ --no-progress --severity HIGH $IMAGE
    # Fail on severe vulnerabilities
    - ./trivy --exit-code 1 --cache-dir .trivycache/ --severity CRITICAL --no-progress $IMAGE
  cache:
    paths:
      - .trivycache/
  # Enables https://docs.gitlab.com/ee/user/application_security/container_scanning/ (Container Scanning report is available on GitLab EE Ultimate or GitLab.com Gold)
  artifacts:
    reports:
      container_scanning: gl-container-scanning-report.json
```

## **Execução manual**

Para rodar o Trivy manualmente, siga os passos abaixo:

### Pré-requisitos
Certifique-se de que o Python e o Trivy estão instalados no ambiente local:

```pip install bandit```

### Comando para análise

- Para gerar um relatório em JSON:

```trivy image mepa-api:latest-dev```

- Para filtrar corrigidos e não afetados:

```trivy image mepa-api:latest-dev --ignore-status fixed,not_affected```

#### Teste do comando:  
![][running]

#### Exemplo de resultados:  
![][results]

---

## **Resultados Obtidos**

### **Relatório de Vulnerabilidades**
A análise identificou os seguintes problemas de segurança:

- **1 Senha hardcoded** detectada, que comprometeria a assinatura criptográfica da aplicação.
- **0 Chaves de API expostas** identificadas.
- **0 Credenciais de usuário expostas** encontradas.

#### Resumo dos Achados:
| Nível de Criticalidade | Ocorrências | Biblioteca |
|-----------------------------|-------------|-----------------------|
| CRITICAL           | 1           | `zlib1g`          |
| HIGH              | 10           | `libexpat1, libgssapi-krb5-2, libk5crypto3, libkrb5-3, libkrb5support0, libldap-2.5-0, libperl5.36, perl, perl-base, perl-modules-5.36` |
| MEDIUM           | 16           | `libexpat1, libgcrypt20, libncursesw6, libpam-modules, libpam-modules-bin, libpam-runtime, libpam0g, libtinfo6, login, ncurses-base, ncurses-bin, passwd` |
| LOW                | 82           | `apt,bash,bsdutils,coreutils,coreutils,gcc-12-base,gcc-12-base,gpgv,libapt-pkg6.0,libblkid1,libc-bin,libc-bin,libc-bin,libc-bin,libc-bin,libc-bin,libc-bin,libc6,libc6,libc6,libc6,libc6,libc6,libc6,libexpat1,libexpat1,libgcc-s1,libgcc-s1,libgcrypt20,libgnutls30,libgssapi-krb5-2,libgssapi-krb5-2,libgssapi-krb5-2,libk5crypto3,libk5crypto3,libk5crypto3,libkrb5-3,libkrb5-3,libkrb5-3,libkrb5support0,libkrb5support0,libkrb5support0,libldap-2.5-0,libldap-2.5-0,libldap-2.5-0,libldap-2.5-0,libmount1,libperl5.36,libperl5.36,libsmartcols1,libsqlite3-0,libstdc++6,libstdc++6,libsystemd0,libsystemd0,libsystemd0,libsystemd0,libudev1,libudev1,libudev1,libudev1,libuuid1,login,login,login,login,mount,passwd,passwd,passwd,passwd,perl,perl,perl-base,perl-base,perl-modules-5.36,perl-modules-5.36,sysvinit-utils,tar,tar,util-linux,util-linux-extra` |
|----|----|----|
| **Total**            | **109**       | **46** |

## **Vulnerabilidades Notáveis**

### **CVE-2023-45853**
- **Biblioteca**: **zlib1g**
- **Versão**: **1:1.2.13.dfsg-1**
- **Criticalidade**: **CRITICAL**
- **Status**: **will_not_fix**
- **Descrição**: O MiniZip no zlib até a versão 1.3 possui um estouro de inteiro e um subsequente **buffer overflow** baseado em heap na função zipOpenNewFileInZip4_64 devido a um nome de arquivo longo, comentário ou campo extra. NOTA: O MiniZip não é uma parte suportada do produto zlib.
- **Recomendação**: Atualizar para uma versão mais recente que não seja vulnerável. **NOTA: O pyminizip até a versão 0.2.6 também é vulnerável porque inclui uma versão afetada do zlib e expõe o código do MiniZip aplicável através de sua API de compressão.**
- **Buffer Overflow**
  - **Descrição**: Um **buffer overflow** ocorre quando um programa tenta armazenar mais dados em um buffer do que ele foi projetado para conter. Isso geralmente resulta em corrupção de memória e pode levar a falhas de segurança. No caso do zlib, o **buffer overflow** é causado por um nome de arquivo longo, comentário ou campo extra. Atacantes podem explorar essa vulnerabilidade para **executar código malicioso**, **escalonar privilégios** ou causar uma **negação de serviço**.

<p align="center">
  <img src=../Images/buffer_overflow.png width="500"/>
</p>


### **TEMP-0841856-B18BAF**
- **Biblioteca**: **bash**
- **Versão**: **5.2.15-2+b7**
- **Criticalidade**: **HIGH**
- **Status**: **affected**
- **Descrição**: Escalonamento de privilégios possível para outro usuário que não seja root. Quando o usuário é root (uid == 0), a variável PS4 não deve ser importada do ambiente, pois poderia ser manipulada de forma maliciosa, expondo comandos sensíveis.
- **Recomendação**: Atualizar para uma versão mais recente que não seja vulnerável ou **bloquear o uso de PS4**. No entanto, o problema ainda poderia ser explorado caso um processo alterasse seu uid para um valor diferente de 0 usando o comando setuid(), mas essa situação seria menos grave.
- **Escalonamento de Privilégios**
  - **Descrição**: O escalonamento de privilégios ocorre quando um usuário comum obtém acesso a recursos ou privilégios que normalmente são restritos a usuários administrativos. No caso do bash, a variável PS4 pode ser manipulada para executar comandos sensíveis com privilégios elevados. Isso pode ser explorado por usuários maliciosos para **obter acesso não autorizado a sistemas ou informações confidenciais**.

<p align="center">
  <img src=../Images/privilage_escalation.png width="500"/>
</p>

### **CVE-2023-52425**
- **Biblioteca**: **libexpat1**
- **Versão**: **2.5.0-1**
- **Criticalidade**: **HIGH**
- **Status**: **affected**
- **Descrição**: libexpat até 2.5.0 permite uma negação de serviço (DoS) porque muitas reanálises completas são necessárias no caso de um token grande para o qual são necessários vários preenchimentos de buffer.
- **Recomendação**: Atualizar para uma versão mais recente que não seja vulnerável.
- **DoS e DDoS**
  - **Descrição**: Um ataque de negação de serviço (DoS) ocorre quando um sistema é sobrecarregado com solicitações maliciosas, resultando em uma interrupção do serviço. No caso da libexpat, um token grande pode causar várias reanálises completas, levando a um DoS. Ataques de negação de serviço distribuídos (DDoS) podem ser ainda mais prejudiciais, pois envolvem múltiplos dispositivos atacando simultaneamente.

<p align="center">
  <img src=../Images/ddos.png width="500"/>
</p>


### **CVE-2024-2236**
- **Biblioteca**: **libgcrypt**
- **Versão**: **1.10.1-3**
- **Criticalidade**: **MEDIUM**
- **Status**: **fix_deferred**
- **Descrição**: Uma falha de canal lateral baseada em temporização foi encontrada na implementação RSA da **libgcrypt**. Esse problema pode permitir que um invasor remoto inicie um ataque do tipo **Bleichenbacher**, que pode levar à **descriptografia de textos cifrados RSA.**
- **Recomendação**: Atualizar para uma versão mais recente que não seja vulnerável.
- **Canal Lateral e Ataques Bleichenbacher**
  - **Descrição**: Um ataque de canal lateral baseado em temporização explora a variação no tempo de execução de um algoritmo criptográfico para inferir informações sobre os dados processados. No caso da libgcrypt, essa vulnerabilidade pode ser explorada para descriptografar textos cifrados RSA. Ataques do tipo Bleichenbacher são uma forma de ataque de canal lateral que visa decifrar mensagens criptografadas

<p align="center">
  <img src=../Images/side_channel.png width="500"/>
</p>

## **Recomendações de Correção**

- **Atualizar Dependências Vulneráveis**: Corrigir as vulnerabilidades críticas e de alta prioridade atualizando as bibliotecas afetadas para versões mais recentes.

- **Adicionar Trivy ao Pipeline**: Manter o Trivy integrado ao pipeline de CI/CD para garantir que novas vulnerabilidades sejam detectadas e corrigidas continuamente.

- **Repensar a versão que a imagem usa de Base pelo Docker**: A imagem base utilizada é a `python:3.11.9-slim-bookworm`, que possui vulnerabilidades críticas herdados da imagem `debian:12-slim`. Recomenda-se pesquisa e escolha de uma imagem base mais segura.
Na data atual as imagens com final `-slim-bookworm` estão na versão `3.14.0a4` e `3.14.0a5`, 3 updates minors a frente da versão utilizada e com correções de segurança para várias das vulnerabilidades encontradas.

- **Realizar um DAST**: Realizar um **teste de segurança dinâmico (DAST)** utilizando as vulnerabilidades encontradas pelas rodadas de SAST implementadas nas sprints anteriores. Isso permitirá identificar possíveis falhas de segurança que podem ser exploradas por atacantes.



[running]: ../Images/trivy_running.png
[results]: ../Images/trivy_results.png
[buffer_overflow]: ../Images/buffer_overflow.png
[privilage_escalation]: ../Images/privilage_escalation.png
[ddos]: ../Images/ddos.png
[side_channel]: ../Images/side_channel.png
