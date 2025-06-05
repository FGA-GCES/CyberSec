# SPRINT 2 - MEPA BLUE TEAM

## Integrantes

- **Ricardo Augusto Valle Maciel** - 180077899 - [@avmricardo](https://github.com/avmricardo)
- **Mateus Orlando Medeiros Ribeiro** - 211062259 - [@MateusPy](https://github.com/MateusPy)
- **Danilo Carvalho Antunes** - 211039312 - [@Danilo-Carvalho-Antunes](https://github.com/Danilo-Carvalho-Antunes)

---

## 🛡️ Atividades de Capacitação Realizadas

### 1 - ⚙️ Estudos da trilha “DevSecOps” – TryHackMe

**Conteúdos abordados:**

- Integração da segurança ao ciclo de desenvolvimento (DevSecOps).
- Práticas de segurança em pipelines CI/CD.
- Ferramentas e técnicas para automação de análise de vulnerabilidades.
- Gerenciamento de dependências e análise de containers.

### 2 - ⚙️ Estudos iniciais sobre Web Application Firewalls (WAF) e OWASP Coraza

**Conteúdos em estudo:**

- Conceitos fundamentais de WAFs: o que são, como funcionam e sua importância em aplicações web modernas.
- Tipos de WAFs: baseados em rede, host e nuvem.
- Introdução ao OWASP Coraza: estrutura, benefícios e funcionalidades principais.
- Integração do Coraza com servidores proxy reverso como Nginx e Caddy.
- Estudo das documentações:
  - [OWASP Coraza Web Application Firewall](https://owasp.org/www-project-coraza-web-application-firewall/)
  - [OWASP Coraza - Quick Start Guide](https://coraza.io/docs/tutorials/quick-start/#requirements)
- Análise de cenários de uso do Coraza na proteção de APIs Python.

---

## 💻 Atividades Práticas Realizadas

### 1 - ✅ Análise das vulnerabilidades encontradas na [issue 1](../relatorios/sprint1.md)

- A partir das vulnerabilidades de nível médio apontadas pelo Bandit no repositório do MEPA API, foi feita uma análise detalhada dos trechos de código afetados.
- Criamos um documento interno com anotações sobre os principais pontos levantados.
- Iniciamos discussões sobre possíveis correções e como o uso de um WAF poderá complementar as proteções oferecidas pelo SAST.

---

## 📈 Próximos Passos

- Iniciar a implementação de um ambiente de testes com o Coraza atuando como WAF para o back-end do MEPA.
- Criar um script com regras padrão (OWASP CRS) para bloquear ataques comuns (XSS, SQLi, LFI etc.).
- Avaliar o impacto do WAF no desempenho da aplicação e sua compatibilidade com as rotas existentes.
- Explorar técnicas para integração dos logs do WAF com sistemas de monitoramento e geração de alertas.
