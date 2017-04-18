---
layout: post
title:  "Dicas rápidas para aumentar sua segurança no Linux"
date:   2017-01-19 00:11:00
disqus: true
categories: security linux
---

Quer saber um pouco mais sobre segurança no Linux? Neste artigo irei dar algumas dicas.

### Primeira dica: Não use o root

Não é necessário e nem seguro ficar sempre logado e executar todos os comandos com o root do sistema.

**Crie o seu próprio usuário:**

{% highlight shell_session %}
$ adduser sergioaugrod
{% endhighlight %}

*Com esse comando será solicitado a senha do novo usuário, é sempre bom colocar uma senha forte, mas não tão forte a ponto de não se lembrar (sempre faço isso).*

**Temos o usuário, mas agora devemos adicioná-lo no *sudoers*, para obtermos permissões administrativas na máquina:**

{% highlight shell_session %}
$ usermod -aG sudo sergioaugrod
{% endhighlight %}

Assim poderemos executar tarefas que exigem privilégios administrativos, como por exemplo parar o serviço do *nginx*:

{% highlight shell_session %}
$ sudo systemctl stop nginx
{% endhighlight %}

*Ao executar um comando administrativo pela primeira vez na sessão, será necessário digitar a senha de seu usuário.*

**Para evitar que alguém autentique com o usuário root, podemos desabilitar a sua permissão de login, editando o arquivo:**

{% highlight shell_session %}
$ sudo vim /etc/ssh/sshd_config
{% endhighlight %}

**Altere a chave:**

```
PermitRootLogin no
```

**Reinicie o ssh:**

{% highlight shell_session %}
$ sudo systemctl restart ssh
{% endhighlight %}

**Para alterar para seu novo usuário:**

{% highlight shell_session %}
$ su - sergioaugrod
{% endhighlight %}

### Segunda dica: Utilize autenticação por chave pública

Com esse passo você não necessitará de senha para acessar seu servidor, mas somente terá acesso mediante seu conjunto de chaves, aumentando consideravelmente sua segurança de acesso.

**Caso não tenha um conjunto de chaves geradas em sua máquina:**

{% highlight shell_session %}
$ ssh-keygen
{% endhighlight %}

**Copie sua chave pública de sua máquina:**

{% highlight shell_session %}
$ ~/.ssh/id_rsa.pub | xclip
{% endhighlight %}

**Crie a pasta .ssh no servidor com as permissões corretas (já utilizando o seu usuário):**

{% highlight shell_session %}
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh
{% endhighlight %}

**Crie o arquivo ~/.ssh/authorized_keys e insira sua chave pública copiada:**

{% highlight shell_session %}
$ vim ~/.ssh/authorized_keys
$ chmod 600 ~/.ssh/authorized_keys
{% endhighlight %}

**Caso queira desabilitar o acesso ao servidor por senha e deixar somente por chave pública, edite o arquivo:**

{% highlight shell_session %}
$ sudo vim /etc/ssh/sshd_config:
{% endhighlight %}

**Altere a linha:**

```
PasswordAuthentication no
```

*Lembrando que com essa alteração você somente terá acesso ao servidor caso esteja no computador que contenha o seu conjunto de chaves.*

**Para finalizar, recarregue o ssh:**

{% highlight shell_session %}
$ sudo systemctl restart ssh
{% endhighlight %}

### Terceira dica: Crie um usuário para executar sua aplicação

Criamos um usuário para acesso administrativo em nosso servidor, mas se vamos utilizar esta mesma máquina para hospedar uma aplicação, isolaremos a execução desta *app* à um usuário com permissões restritas.

**Crie o usuário:**

{% highlight shell_session %}
$ sudo adduser ghost
{% endhighlight %}

**Crie uma pasta para sua aplicação:**

{% highlight shell_session %}
$ sudo mkdir /var/www/ghost
{% endhighlight %}

*É comum deixar as aplicações no diretório /var/www.*

**Dê permissão na pasta para o usuário:**

{% highlight shell_session %}
$ sudo chown -R ghost:ghost /var/www/ghost
{% endhighlight %}

*Assim o usuário da aplicação poderá executa-lá.*

**Execute sua aplicação com o usuário criado:**

{% highlight shell_session %}
$ su - ghost
$ cd /var/www/ghost
$ npm start
{% endhighlight %}

### Quarta dica: Mantenha seu sistema atualizado

**Por último, a dica mais fácil, mas não menos importante, sempre atualize seu sistema e suas dependências:**

{% highlight shell_session %}
$ apt-get update
$ apt-get upgrade
{% endhighlight %}

### Concluindo

Essas foram algumas dicas bem básicas para aumentar um pouco a segurança de seu Linux, todos os comandos foram executados em um Debian, mas podem ser facilmente portados para outras distribuições. Obrigado e até a próxima!
