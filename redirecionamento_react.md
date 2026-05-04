A localização dos arquivos e configurações podem variar e certas pastas podem estar presentes ou não dependendo da distribuição Linux que você estiver utilizando. No final deste tutorial, há uma seção de configuração específica para a versão "tradicional" do Nginx.

Primeiro, vamos verificar as pastas corretas do Nginx utilizando o comando `find`.
```bash
root@03e4c87dc48e:/# find /etc/nginx -type f -name "*.conf"
/etc/nginx/conf.d/default.conf
/etc/nginx/sites-available/default
```
Ao acessar o arquivo `default.conf`, você irá se deparar com algo parecido com isso:
```ngnix
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
Dentro do "contexto" `location`, você servirá o arquivo principal `index.html` por meio de uma diretiva chamada `try_files`. Adicione a diretiva e após isso salve o arquivo.
```ngnix
	location / {
		try_files $uri $uri/ /index.html;
	}
```
Para verificar se a diretiva foi adicionada corretamente utilize o comando `nginx -t`, se tudo correr bem, vermos as seguintes mensagens no terminal:
```bash
root@03e4c87dc48e:/# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
Após isso, reinicie o Nginx. Se você estiver em um contêiner, basta utilizar o comando `docker restart nome_do_conteiner`  ou se você estiver em uma máquina real, utilize o comando `systemctl restart nginx`.
>[!question] Cadê o `sites_enabled` e `sites_avaliable`?
>Os contêineres do Nginx não possuem as pastas `sites-avaliable`  e `sites-enabled` por questões de simplicidade. O processo para as versões "tradicionais" possuem alguns passos um pouco mais específicos. Comece adicionando um arquivo de configuração `nginx` do serviço que você deseja subir:
>```bash
>sudo nano /etc/nginx/sites-available/meu-app-react
>```
>Em seguida, adicione a diretiva `try_files` assim como visto anteriormente, após isso crie um link simbólico entre o `sites-avaliable` e `sites_enabled`:
>```bash
>sudo ln -s /etc/nginx/sites-available/meu-app-react /etc/nginx/sites-enabled/
>```
>E por fim, verifique se todas as pastas estão inclusas no arquivo `nginx.conf`
>```nginx
>include /etc/nginx/conf.d/*.conf;
>include /etc/nginx/sites-enabled/*;
>```

Versão do Nginx utilizada: `nginx/1.29.8`
Referências:
- https://hub.docker.com/_/nginx
- https://docs.nginx.com/nginx/admin-guide/web-server/serving-static-content/
- https://nginx.org/en/docs/http/ngx_http_core_module.html#try_files