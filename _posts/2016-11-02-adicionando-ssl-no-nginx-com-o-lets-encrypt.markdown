---
layout: post
title:  "Adicionando SSL no NGINX com o Let's Encrypt"
date:   2016-11-02 21:30:00
categories: security
---

O **Let's Encrypt** é uma forma fácil, automatizada e gratuita de se inserir **SSL** em uma aplicação *web*. A utilização do **SSL** é bem importante quando se há autenticação, tráfego de dados privados ou até mesmo para ser melhor colocado no *ranking* do *Google*.

Neste artigo vou demonstrar como gerar e adicionar o **SSL** no **NGINX** com a ferramenta **Let's Encrypt**. Irei utilizar o sistema operacional *Debian* para executar os comandos, mas estes podem ser facilmente modificados para serem executados em qualquer *distro*.

#### Instalando o Let's Encrypt:

Clone o projeto no *github* e redirecione para o caminho **/opt/letsencrypt**:

{% highlight shell_session %}
$ sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt
{% endhighlight %}

É necessário ter o *git* instalado, caso não tenha:

{% highlight shell_session %}
$ sudo apt-get install git
{% endhighlight %}

#### Preparando o NGINX para ser validado:

O **Let's Encrypt** valida se o domínio realmente é seu, então para isso é necessário adicionar uma regra no seu *site* do **NGINX**. Adicione o **location** *^/.well-known* no seu site **(/etc/nginx/sites-enabled/yoursite)**, como por exemplo:

{% highlight nginx %}
server {
    listen 80;
    server_name your-domain.com.br;

    location ~ ^/.well-known {
        root /var/www/yoursite;
    }

    location / {
        return 301 https://www.$server_name$request_uri;
    }
}
{% endhighlight %}

Este **location** será requisitado pelo **Let's Encrypt** para confirmar sua identidade. Lembrando que você deve substituir o **root** e o **server_name**.

Após adicionar o *well-known*, reinicie o seu **NGINX**:

{% highlight shell_session %}
$ sudo systemctl restart nginx
{% endhighlight %}

*É bom lembrar que o seu domínio deve estar apontando para sua aplicação para obter sucesso com o **SSL**.*

#### Gerando o SSL com o Let's Encrypt:

Substitua no comando abaixo, o caminho **/var/www/yoursite** pelo diretório raiz do seu site no **NGINX**, e também o **yourdomain.com.br** e **www.yourdomain.com.br** pelo seu domínio:

{% highlight shell_session %}
$ sudo /opt/letsencrypt/letsencrypt-auto certonly -a webroot --webroot-path=/var/www/yoursite -d yourdomain.com.br -d www.yourdomain.com.br
{% endhighlight %}

Neste processo irá ser solicitado seu *e-mail*, para caso necessite da recuperação de seu certificado.

#### Adicionando o certificado em sua aplicação:

Após o certificado ser gerado com sucesso, altere novamente o seu arquivo de regras do seu site **(/etc/nginx/sites-enabled/yoursite)**, adicionando mais um **server**, desta vez o de **ssl**:

{% highlight nginx %}
server {
    listen 443 ssl;
    server_name yourdomain.com.br www.yourdomain.com.br;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }

    location ~ ^/.well-known {
        root /var/www/yoursite;
    }

    ssl_certificate /etc/letsencrypt/live/yourdomain.com.br/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com.br/privkey.pem;
}
{% endhighlight %}

*Lembrando que você deve alterar o **location /** com as configurações específicas da sua aplicação web.*

Reinicie novamente o seu **NGINX**:


{% highlight shell_session %}
$ sudo systemctl restart nginx
{% endhighlight %}

Entre em seu domínio utilizando o **https** e veja se o processo ocorreu com sucesso.

#### Conferindo a qualidade do seu SSL:

Altere *example.com* pelo seu domínio:

<https://www.ssllabs.com/ssltest/analyze.html?d=example.com>

#### Melhorando a qualidade do seu certificado:

É importante validar as cifras utilizadas, limitar a versão do protocolo **SSL**, entre outras coisas. Para isso, recomendo a leitura do seguinte tópico na wiki da Mozilla, [Server Side TLS](https://wiki.mozilla.org/Security/Server_Side_TLS). Existe também o [Mozilla SSL Configuration Generator](https://mozilla.github.io/server-side-tls/ssl-config-generator), um gerador de configuração **SSL** para diversos servidores de aplicação.

#### Renovando seu certificado com crontab:

O certificado gerado é válido por 3 meses, para facilitar a renovação, você pode criar um *cronjob* para fazer este trabalho:

{% highlight shell_session %}
$ crontab -e
{% endhighlight %}

Adicione no final do arquivo:

```
0 0 1 */2 * /opt/letsencrypt/letsencrypt-auto renew --quiet --no-self-upgrade
0 0 1 */2 * systemctl reload nginx
```

### Concluindo:

Neste artigo foi demonstrando a geração do **SSL** para o **NGINX**, mas este mesmos passos podem ser facilmente executados em qualquer servidor de aplicação, com algumas modificações. Lembrando que existem outros comandos específicos da ferramenta **Let's Encrypt**, como **letsencrypt-apache** que faz todo o trabalho pra você no caso do *Apache*, mas tentei demonstrar a forma genérica, que pode servir para outros servidores.

### Referências e Links:

* <https://letsencrypt.org>
* <https://github.com/certbot/certbot>
* <https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04>
* <https://www.nginx.com/blog/free-certificates-lets-encrypt-and-nginx>
