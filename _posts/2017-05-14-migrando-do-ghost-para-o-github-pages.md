---
title: Migrando do Ghost para o GitHub Pages
layout: post
date: '2017-05-14 09:29:37'
categories: offtopic
---

Há pouco mais de 1 mês o blog ficava em um cloud e rodava com a plataforma de blog **Ghost**, e ficou assim por quase 1 ano de existência. O blog é pequeno, tem poucos acessos, e então eu não notava nenhuma vantagem em se ter um cloud, e ter que ficar atualizando a versão do Ghost, além de ter que acabar fazendo algumas alterações no debian/nginx/node de vez em quando. 

A única grande vantagem que eu sempre enxergava era a possibilidade de utilizar o Let's Encrpy para o SSL. Mas no final, acabei migrando todo o blog para **GitHub Pages**, utilizando o **Jekyll** para fazer a mágica de transformar os arquivos *markdown* em um blog.

#### Tema

Uma coisa que gostava no Ghost era o visual, e queria ter algo parecido no Jekyll, foi aí que encontrei o <https://github.com/rosario/kasper>, um tema baseado no Casper do Ghost, utilizei ele para dar o bootstrap no projeto.

#### Administrando

O painel administrativo do Ghost também era bem completo, queria algo parecido para poder gerenciar os posts (arquivos *markdown*), achei o <https://github.com/jekyll/jekyll-admin> para fazer este trabalho. Assim eu vou escrevendo os posts localmente, e quando acabo eu faço um push para o github.

#### SSL

Como não consigo mais utilizar o **Let's Encrypt**, tive que partir para outras alternativas, uma que encontrei foi deixar o DNS do blog no **Cloudflare** e utilizar o **SSL** gratuíto deles.

#### Finalizando

Deu um bom trabalho transformar todos os *posts* em arquivos *markdown*, e deixar o blog redondo com a utilização alguns plugins, mas o resultado final foi bem legal. Para finalizar quero listar todas as vantagens e desvantagens que eu percebi desta mudança:

**Vantagens:**

- É grátis, não tenho mais o custo de um cloud;
- Não preciso gerenciar atualizações no sistema operacional e nem preocupar com a segurança da máquina;
- Não preciso atualizar o Ghost e nem o Node.js;
- Não preciso fazer nenhuma configuração de nginx, systemd, nodemon, etc;
- Nunca vou precisar me preocupar com reconfiguração de tudo isso acima.

**Desvantagens:**

- Não consigo escrever os rascunhos quando só tenho o browser como ferramenta;
- Não posso mais o utilizar o Let's Encrypt para o SSL.

#### Links

- <https://github.com/sergioaugrod/sergioaugrod.github.io>
- <https://pages.github.com>
- <https://jekyllrb.com>
- <https://github.com/rosario/kasper>
- <https://github.com/jekyll/jekyll-admin>
- <https://www.cloudflare.com>
- <https://letsencrypt.org>