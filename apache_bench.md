# Testes de Carga com Apache Bench

## Introdução

Quando você implanta uma aplicação React em produção, é fundamental entender como o servidor se comporta sob diferentes níveis de carga. Um servidor que funciona perfeitamente com 10 usuários simultâneos pode começar a apresentar lentidão ou falhas com 1000 usuários. É nesse contexto que ferramentas de teste de carga como o **Apache Bench** entram em jogo.

Apache Bench (frequentemente chamado de `ab`) é uma ferramenta de linha de comando incluída no pacote Apache HTTP Server que permite simular múltiplas requisições HTTP para um servidor web. Ela é simples, direta e perfeita para testes iniciais de performance. Com o Apache Bench, você pode gerar requisições simultâneas, medir tempos de resposta, identificar gargalos e verificar como sua aplicação React se comporta sob pressão.

Neste tutorial, vamos utilizar o Apache Bench (versão `2.3`) executado dentro de um contêiner do `httpd` (versão `2.4.66`) para realizar testes de carga contra uma aplicação React servida pelo Nginx. Os testes nos ajudarão a entender métricas importantes como tempo médio de resposta, requisições por segundo e taxa de sucesso.

## Preparando o Ambiente

O Apache Bench já vem instalado por padrão no contêiner do Apache HTTP Server. Ao iniciar o contêiner `httpd`, você terá acesso imediato à ferramenta `ab` sem necessidade de instalação adicional.

Para acessar o contêiner e começar a utilizar o Apache Bench, utilize o comando:

```bash
docker exec -it nome_do_conteiner_httpd /bin/bash
```

Dentro do contêiner, você pode verificar a versão do Apache Bench utilizando:

```bash
ab -V
```

Se tudo estiver correto, você verá algo parecido com isto:

```bash
root@a3f2b1c9d5e6:/# ab -V
This is ApacheBench, Version 2.3
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
```

## Principais Parâmetros do Apache Bench

Antes de executarmos os testes, é importante entender quais parâmetros utilizaremos e para que servem cada um deles:

- `-n <número>` — Define o número total de requisições que serão feitas ao servidor. Por exemplo, `-n 1000` significa que 1000 requisições serão enviadas.

- `-c <número>` — Define o número de requisições simultâneas (concorrência). Por exemplo, `-c 50` significa que até 50 requisições serão enviadas ao mesmo tempo. Requisições que chegarem ao número total (`-n`) serão enfileiradas.

- `-t <segundos>` — Define um tempo limite em segundos para o teste. Se o servidor não responder em tempo hábil, o teste é interrompido.

- `-H "Header: value"` — Permite adicionar cabeçalhos HTTP customizados à requisição. Útil quando você precisa testar autenticação ou outros comportamentos específicos.

- `-p <arquivo>` — Especifica um arquivo contendo dados POST que serão enviados junto com as requisições. Útil para testar endpoints que aceitam dados no corpo da requisição.

- `-g <arquivo>` — Exporta os resultados em formato `.tsv` (values separated by tabs), que pode ser importado em ferramentas como Excel ou usados para gerar gráficos.

## Executando Testes de Carga

Realizamos testes de carga contra uma aplicação React servida pelo Nginx em um ambiente containerizado. Os testes simulam diferentes cenários de carga para avaliar a performance e a estabilidade do servidor.

>[!NOTE]
>**Sobre o hostname `servidor-web`**
>
>Os testes foram originalmente desenvolvidos na máquina local usando `localhost:8080`. No entanto, para permitir que o contêiner do Apache Bench (httpd) se comunicasse com o contêiner do Nginx, foi necessário criar uma rede Docker personalizada. Nesta configuração, o contêiner do Nginx recebeu o hostname `servidor-web`, que é resolvido automaticamente pela rede Docker interna. Portanto, ao executar os testes, a URL utilizada foi `http://servidor-web/` em vez de `http://localhost:8080/`. Se você estiver executando os testes em seu próprio ambiente, adapte o hostname/porta conforme necessário (ex: `http://localhost:8080/` ou `http://seu-servidor.local/`).

### Teste 1: Carga Moderada

Este teste simula um cenário com tráfego moderado, onde múltiplos usuários acessam a aplicação simultaneamente:

```bash
ab -n 10000 -c 50 http://servidor-web/
```

Este comando envia **10.000 requisições** com uma concorrência de **50 requisições simultâneas** para a URL raiz da aplicação. O resultado obtido foi:

```
Server Software:        nginx/1.30.0
Server Hostname:        servidor-web
Server Port:            80

Document Path:          /
Document Length:        470 bytes

Concurrency Level:      50
Time taken for tests:   0.766 seconds
Complete requests:      10000
Failed requests:        0
Total transferred:      7030000 bytes
HTML transferred:       4700000 bytes
Requests per second:    13062.85 [#/sec] (mean)
Time per request:       3.828 [ms] (mean)
Time per request:       0.077 [ms] (mean, across all concurrent requests)
Transfer rate:          8967.95 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    1   0.6      1       4
Processing:     1    2   0.8      2       6
Waiting:        0    2   0.7      2       5
Total:          2    4   1.2      4       8

Percentage of the requests served within a certain time (ms)
  50%      4
  66%      4
  75%      5
  80%      5
  90%      5
  95%      6
  98%      6
  99%      6
 100%      8 (longest request)
```

**Análise:** O servidor respondeu excelentemente sob carga moderada. Com uma taxa de **13.062 requisições por segundo** e tempo médio de resposta de apenas **3.828 ms**, o servidor manteve uma performance consistente. Nenhuma requisição falhou, indicando estabilidade total. O percentil 95% ficou em apenas 6ms, o que demonstra que 95% das requisições foram respondidas muito rapidamente.

### Teste 2: Carga no Pico de Stress

Este teste simula um cenário de pico de tráfego, onde o servidor enfrenta uma quantidade significativa de requisições simultâneas:

```bash
ab -n 50000 -c 100 http://servidor-web/
```

Este comando envia **50.000 requisições** com **100 requisições simultâneas**. O resultado obtido foi:

```
Server Software:        nginx/1.30.0
Server Hostname:        servidor-web
Server Port:            80

Document Path:          /
Document Length:        470 bytes

Concurrency Level:      100
Time taken for tests:   3.065 seconds
Complete requests:      50000
Failed requests:        0
Total transferred:      35150000 bytes
HTML transferred:       23500000 bytes
Requests per second:    16311.15 [#/sec] (mean)
Time per request:       6.131 [ms] (mean)
Time per request:       0.061 [ms] (mean, across all concurrent requests)
Transfer rate:          11197.99 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    2   0.9      2       8
Processing:     1    4   1.2      3      12
Waiting:        0    3   1.0      3       9
Total:          3    6   1.8      6      17

Percentage of the requests served within a certain time (ms)
  50%      6
  66%      6
  75%      7
  80%      7
  90%      9
  95%     10
  98%     11
  99%     12
 100%     17 (longest request)
```

**Análise:** Mesmo sob pressão intensa, o servidor manteve uma performance notável. A taxa de requisições por segundo aumentou para **16.311 RPS**, indicando que o servidor conseguiu escalar bem com mais concorrência. O tempo médio de resposta foi de **6.131 ms**, mantendo-se abaixo de 10ms. Novamente, nenhuma requisição falhou. O percentil 99% ficou em 12ms, mostrando que até mesmo na pior situação (1% das requisições mais lentas), o tempo permaneceu aceitável.

### Teste 3: Carga Limite

Este teste coloca o servidor no seu limite máximo de capacidade, simulando um pico extremo de tráfego:

```bash
ab -n 100000 -c 500 http://servidor-web/
```

Este comando envia **100.000 requisições** com **500 requisições simultâneas**. O resultado obtido foi:

```
Server Software:        nginx/1.30.0
Server Hostname:        servidor-web
Server Port:            80

Document Path:          /
Document Length:        470 bytes

Concurrency Level:      500
Time taken for tests:   6.875 seconds
Complete requests:      100000
Failed requests:        0
Total transferred:      70300000 bytes
HTML transferred:       47000000 bytes
Requests per second:    14546.42 [#/sec] (mean)
Time per request:       34.373 [ms] (mean)
Time per request:       0.069 [ms] (mean, across all concurrent requests)
Transfer rate:          9986.46 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0   16   4.2     15      41
Processing:     4   19   5.2     17      52
Waiting:        0   13   4.1     12      42
Total:         16   34   7.9     32      71

Percentage of the requests served within a certain time (ms)
  50%     32
  66%     35
  75%     38
  80%     40
  90%     45
  95%     51
  98%     57
  99%     60
 100%     71 (longest request)
```

**Análise:** Mesmo com 500 requisições simultâneas, o servidor continuou respondendo sem falhas. A taxa de requisições por segundo foi de **14.546 RPS**, ligeiramente menor que o teste anterior, o que é esperado dado o aumento massivo de concorrência. O tempo médio de resposta subiu para **34.373 ms**, ainda dentro de um limite aceitável para a maioria das aplicações. Nenhuma requisição foi rejeitada, demonstrando que a infraestrutura configurada é robusta e confiável mesmo sob condições extremas.

## Interpretando os Resultados

Os resultados obtidos revelam importantes métricas sobre o comportamento do servidor:

**Requests per second (RPS)** — Este número indica quantas requisições o servidor consegue processar a cada segundo. Nos nossos testes, observamos uma variação entre 13.062 e 16.311 RPS dependendo da concorrência. Quanto maior este valor, melhor a capacidade de processamento do servidor.

**Time per request** — Este valor representa o tempo médio que o servidor leva para processar uma requisição completa. Nos testes, variou de 3.828ms a 34.373ms. Em aplicações web modernas, tempos abaixo de 100ms são geralmente considerados excelentes.

**Failed requests** — Este é talvez o métrica mais crítica. Em todos os nossos testes, o número de requisições falhadas foi **zero**, indicando que o servidor manteve 100% de disponibilidade mesmo sob carga extrema.

**Percentis de latência** — Estes valores mostram o tempo de resposta em diferentes percentis. Por exemplo, no teste de carga moderada, o percentil 95% foi 6ms, significando que 95% das requisições foram respondidas em até 6ms. Percentis mais altos (como 99%) indicam os piores cenários de latência, que no nosso caso permaneceram sempre aceitáveis.

**Conclusão dos Testes**

A aplicação React servida pelo Nginx demonstrou uma performance excepcional em todos os cenários testados. O servidor manteve alta disponibilidade, baixa latência e escalabilidade consistente, mesmo quando submetido a condições extremas de carga. Esses resultados indicam que a configuração realizada anteriormente (confira o tutorial sobre redirecionamento React com Nginx) é adequada para lidar com aplicações em ambiente de produção.

## Especificações da Máquina Utilizada

Os testes foram realizados em um notebook HP HP 256R 15.6 inch G9 com as seguintes configurações:

**Hardware:**
- Processador: Intel Core i3-1315U (6 núcleos / 8 threads)
- Memória RAM: 24GB DDR4 3200 MT/S
- Armazenamento: SSD

**Sistema Operacional:**
- Distribuição: Debian Linux 13
- Kernel: Linux (6.12.74+deb13+1-amd64)

**Ambiente de Containerização:**
- Docker (com rede personalizada para comunicação entre contêineres)

Estes resultados demonstram que uma máquina de especificações modestas é suficiente para executar testes de carga realistas em uma aplicação React servida pelo Nginx, validando a eficiência tanto da aplicação quanto da infraestrutura configurada.

Versão do Apache Bench utilizada: `2.3`  
Versão do httpd utilizada: `2.4.66`  
Versão do Nginx testada: `nginx/1.30.0`

## Referências

- https://httpd.apache.org/docs/2.4/programs/ab.html
- https://en.wikipedia.org/wiki/ApacheBench
- https://hub.docker.com/_/httpd
- https://www.digitalocean.com/community/tutorials/how-to-use-apache-bench-to-perform-load-testing-on-a-web-server
- https://www.cloudflare.com/learning/performance/load-testing/