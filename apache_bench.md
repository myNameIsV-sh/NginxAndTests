# Testes de Carga com Apache Bench

## Introdução

Quando você implanta uma aplicação React em produção, é fundamental entender como o servidor se comporta sob diferentes níveis de carga. Um servidor que funciona perfeitamente com 10 usuários simultâneos pode começar a apresentar lentidão ou falhas com 1000 usuários. É nesse contexto que ferramentas de teste de carga como o **Apache Bench** entram em jogo.

Apache Bench ou `ab`, é uma ferramenta de linha de comando incluída no pacote Apache HTTP Server que permite simular múltiplas requisições HTTP para um servidor web. Ela é simples, direta e perfeita para testes iniciais de performance. Com o Apache Bench, você pode gerar requisições simultâneas, medir tempos de resposta, identificar gargalos e verificar como sua aplicação React se comporta sob pressão.

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

Vamos simular alguns cenários de teste contra uma aplicação React fictícia servida pelo Nginx. Considere que sua aplicação está disponível em `http://meu-app-react.local`.

### Teste 1: Requisições Simples com Concorrência Baixa

Este é um teste básico para verificar se o servidor responde corretamente:

```bash
ab -n 100 -c 10 http://meu-app-react.local/
```

Este comando envia **100 requisições** com uma concorrência de **10 requisições simultâneas** para a URL raiz da aplicação. O Apache Bench exibirá estatísticas como tempo médio de resposta, requisições por segundo e percentis de latência.

### Teste 2: Teste de Carga Moderada

Para simular uma aplicação sob carga moderada, aumentamos tanto o número total de requisições quanto a concorrência:

```bash
ab -n 1000 -c 50 http://meu-app-react.local/
```

Este teste envia **1000 requisições** com **50 requisições simultâneas**. Este cenário é mais realista e ajuda a identificar como o servidor se comporta com múltiplos usuários simultâneos.

### Teste 3: Teste de Carga Pesada

Para colocar o servidor sob pressão mais intensa:

```bash
ab -n 5000 -c 100 http://meu-app-react.local/
```

Este teste envia **5000 requisições** com **100 requisições simultâneas**. Este cenário simula uma situação de pico de tráfego e pode revelar comportamentos problemáticos como timeouts ou degradação severa de performance.

### Teste 4: Teste de uma Rota Específica

Você também pode testar rotas específicas da sua aplicação React:

```bash
ab -n 500 -c 25 http://meu-app-react.local/sobre
```

Este teste foca na rota `/sobre`, permitindo que você compare a performance de diferentes páginas da aplicação.

### Teste 5: Exportando Resultados para Análise

Para salvar os resultados do teste em um arquivo que possa ser analisado posteriormente:

```bash
ab -n 1000 -c 50 -g resultados_teste.tsv http://meu-app-react.local/
```

O arquivo `resultados_teste.tsv` será criado com dados tabulares que podem ser importados em ferramentas de visualização para análise mais detalhada.

## Interpretando os Resultados

Ao executar um teste, o Apache Bench exibe um relatório com várias métricas importantes. As principais são:

- **Requests per second (RPS)** — Número médio de requisições que o servidor processa por segundo. Quanto maior, melhor.

- **Time per request** — Tempo médio que o servidor leva para processar uma requisição. Expresso em milissegundos (ms).

- **Failed requests** — Número de requisições que resultaram em erro. Idealmente, este número deve ser zero.

- **Percentis de latência** — Mostram o tempo de resposta em diferentes percentis (50%, 90%, 99%). Por exemplo, o percentil 95% significa que 95% das requisições foram respondidas em até esse tempo.


Versão do Apache Bench utilizada: `2.3`  
Versão do httpd utilizada: `2.4.66`

## Referências

- https://httpd.apache.org/docs/2.4/programs/ab.html
- https://en.wikipedia.org/wiki/ApacheBench
- https://hub.docker.com/_/httpd
- https://www.digitalocean.com/community/tutorials/how-to-use-apache-bench-to-perform-load-testing-on-a-web-server
- https://www.cloudflare.com/learning/performance/load-testing/