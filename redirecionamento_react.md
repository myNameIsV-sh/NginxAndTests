# Introdução

Aplicações React modernas utilizam bibliotecas como [React Router](https://reactrouter.com/) para gerenciar rotas no lado do cliente (client-side routing). Isso significa que toda a navegação acontece no navegador, sem recarregar a página ou fazer requisições ao servidor para cada mudança de rota. Quando o usuário clica em um link como `/sobre` ou `/produtos`, o React Router intercepta essa ação e atualiza o conteúdo da página sem enviar uma requisição HTTP para o servidor. Toda a lógica de roteamento acontece em JavaScript, executado no browser.

No momento da construção de uma aplicação React com `npm run build`, o resultado é um diretório contendo um único arquivo `index.html` e alguns arquivos estáticos (JavaScript, CSS, imagens), o resultado é parecido com isso:

```text
meu-app-react/
├── src/
├── public/
├── dist/                    ← Isso vai para o Nginx
│   ├── index.html
│   ├── assets/
│   │   ├── js/
│   │   └── css/
```

E é nesse momento que os problemas começam a surgir. O gerenciamento das rotas dentro do `index.html` é feito pelo próprio React Router, porém o Nginx não compreende isso. Ele espera servir um arquivo chamado `index.html`, `sobre.html` ou `produtos.html` e quando um usuário tenta acessar diretamente a URL `https://seuapp.com/sobre` ou recarrega a página nesse caminho, o servidor Nginx procura um arquivo literal chamado `sobre` ou uma pasta chamada `sobre` que não existe. Naturalmente, o Nginx retorna um erro **404 Not Found**.

Para resolver essa situação, utilizamos a diretiva `try_files` do Nginx. Sempre que chegar uma requisição para uma rota desconhecida, em vez de retornar o erro 404, o Nginx tenta procurar o arquivo solicitado (`$uri`), depois a pasta (`$uri/`), e se nenhum existir, redireciona para `index.html`. Dessa forma, o React Router carrega, analisa a URL atual e renderiza o componente correto.

## Configurando o Nginx

A localização dos arquivos e configurações pode variar dependendo da distribuição Linux que você estiver utilizando. No final deste tutorial, há uma seção de configuração específica para a versão "tradicional" do Nginx.

Primeiro, vamos verificar as pastas corretas do Nginx utilizando o comando `find`:

```bash
root@03e4c87dc48e:/# find /etc/nginx -type f -name "*.conf"
/etc/nginx/conf.d/default.conf
/etc/nginx/sites-available/default
```

Ao acessar o arquivo `default.conf`, você encontrará algo parecido com isto:

```nginx
server {
    listen       80;
    listen  [::]:80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

    # proxy the PHP scripts to Apache listening on 127.0.0.1:80
    #
    #location ~ \.php$ {
    #    proxy_pass   http://127.0.0.1;
    #}

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}
```

Dentro do "contexto" `location`, você serve o arquivo principal `index.html` por meio da diretiva `try_files`. Adicione a diretiva e após isso salve o arquivo:

```nginx
location / {
	root /app/dist
    try_files $uri $uri/ /index.html;
}
```

> [!IMPORTANT]
Ajuste o caminho do root conforme necessário. O caminho `/app/dist` é apenas um exemplo. Você deve substituir pelo caminho real onde se encontra o diretório `dist` do seu projeto. Exemplos comuns incluem `/home/usuario/meu-app-react/dist` ou `/var/www/meu-app/dist`.

Para verificar se a diretiva foi adicionada corretamente, utilize o comando `nginx -t`. Se tudo correr bem, veremos as seguintes mensagens no terminal:

```bash
root@03e4c87dc48e:/# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

Após isso, basta recarregar as novas configurações por meio do comando `nginx -s reload`. Enquanto o Nginx estiver em execução, as novas atualizações serão efetivadas gradativamente. No entanto, em ambientes conteinerizados como Docker, o processo é diferente. Dentro de um contêiner, o Nginx é frequentemente o processo principal (PID 1), e sinais do sistema operacional não funcionam da mesma forma que em uma máquina real. Por isso, ao usar Docker, utilize o comando `docker restart nome_do_conteiner` para reiniciar completamente o contêiner e garantir que as configurações sejam carregadas corretamente. Se você estiver em uma máquina real com Nginx instalado diretamente no sistema operacional, utilize o comando `systemctl restart nginx`.

>[!NOTE] 
>**Cadê o `sites_enabled` e `sites_available`?**
>
>Os contêineres do Nginx não possuem as pastas `sites-available` e `sites-enabled` por questões de simplicidade. O processo para as versões "tradicionais" possuem alguns passos um pouco mais específicos. Comece adicionando um arquivo de configuração `nginx` do serviço que você deseja subir:
>
>```bash
>sudo nano /etc/nginx/sites-available/meu-app-react
>```
>
>Em seguida, adicione a diretiva `try_files` assim como visto anteriormente. Após isso, crie um link simbólico entre o `sites-available` e `sites-enabled`:
>
>```bash
>sudo ln -s /etc/nginx/sites-available/meu-app-react /etc/nginx/sites-enabled/
>```
>
>E por fim, verifique se todas as pastas estão incluídas no arquivo `nginx.conf`:
>
>```nginx
>include /etc/nginx/conf.d/*.conf;
>include /etc/nginx/sites-enabled/*;
>```

Versão do Nginx utilizada: `nginx/1.29.8`

## Referências
- https://hub.docker.com/_/nginx
- https://nginx.org/en/docs/beginners_guide.html
- https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/
- https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files
- https://linux.die.net/man/8/nginx
- https://www.slingacademy.com/article/nginx-try_files-directive-explained-with-examples/#introduction-to-nginx-try_files