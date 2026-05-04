# React Router com Nginx e Testes de Carga

Bem-vindo! Este repositório contém dois tutoriais complementares para implantar e testar uma aplicação React em produção usando Nginx e ferramentas de teste de carga.

### 1. [Redirecionamento React com Nginx](./redirecionamento_react.md)

Um guia completo para configurar o Nginx de forma que aplicações React com React Router funcionem corretamente em produção.

**Tópicos cobertos:**
- Como o React Router funciona no lado do cliente (client-side routing)
- Por que o Nginx entra em conflito com SPAs (Single Page Applications)
- Configuração da diretiva `try_files` para resolver rotas desconhecidas
- Diferenças entre containerização (Docker) e máquinas reais
- Configuração para versões "tradicionais" com `sites-available` e `sites-enabled`

**Versões utilizadas:**
- Nginx: `1.29.8`

---

### 2. [Testes de Carga com Apache Bench](./apache_bench.md)

Um tutorial prático sobre como usar Apache Bench para avaliar a performance da sua aplicação React servida pelo Nginx.

**Tópicos cobertos:**
- Introdução ao Apache Bench e sua importância em testes de performance
- Preparação do ambiente com Docker
- Principais parâmetros do Apache Bench (`-n`, `-c`, `-t`, `-H`, `-p`, `-g`)
- Três cenários de teste (carga moderada, pico de stress, carga limite)
- Interpretação de resultados e métricas importantes

**Resultados obtidos:**
- **Carga Moderada:** 13.062 RPS com tempo médio de 3.828ms
- **Pico de Stress:** 16.311 RPS com tempo médio de 6.131ms
- **Carga Limite:** 14.546 RPS com tempo médio de 34.373ms
- Taxa de sucesso: **100%** em todos os testes (zero requisições falhadas)

**Versões utilizadas:**
- Apache Bench: `2.3`
- httpd: `2.4.66`
- Nginx testado: `1.30.0`

**Hardware utilizado:**
- Processador: Intel Core i3-1315U (6 núcleos / 8 threads)
- Memória RAM: 24GB DDR4 3200 MT/S
- SO: Debian Linux 13
- Máquina: HP 256R 15.6 inch G9

---

## Fluxo Recomendado

1. **Comece com o tutorial de Redirecionamento React com Nginx** — Configure sua aplicação para servir corretamente com Nginx
2. **Implante sua aplicação React** — Use o `npm run build` e configure os arquivos estáticos
3. **Teste com Apache Bench** — Use o segundo tutorial para avaliar a performance sob diferentes cargas

---

## 📋 Resumo Rápido

| Aspecto | Detalhes |
|---------|----------|
| **Problema Principal** | Nginx retorna 404 para rotas do React Router ao acessar diretamente |
| **Solução** | Usar a diretiva `try_files` para redirecionar requisições desconhecidas para `index.html` |
| **Performance Observada** | Excelente em todos os cenários (até 16.311 RPS com latência baixa) |
| **Disponibilidade** | 100% de sucesso mesmo sob carga extrema (500 requisições simultâneas) |
| **Conclusão** | A configuração é adequada para produção em máquinas de especificações modestas |

---

## 🔗 Links Rápidos

- [Documentação do Nginx](https://nginx.org/en/docs/)
- [React Router Documentation](https://reactrouter.com/)
- [Apache HTTP Server](https://httpd.apache.org/)
- [Docker Documentation](https://docs.docker.com/)

---

## Observações Importantes

- Sempre valide suas configurações Nginx com `nginx -t` antes de recarregar
- Em ambientes Docker, use `docker restart` em vez de sinais do sistema
- Os testes de carga devem ser executados contra o servidor em produção ou em ambiente muito similar
- Monitore métricas como RPS, latência percentil 95% e taxa de falhas
- Uma máquina de especificações modestas é suficiente para testes realistas

---

## Considerações Finais

Estes tutoriais foram criados como material educativo e documentação de boas práticas para implantar aplicações React em produção com segurança e performance.

Para dúvidas específicas sobre Nginx ou Apache Bench, consulte a documentação oficial de cada projeto.

---
