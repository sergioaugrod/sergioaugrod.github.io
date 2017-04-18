---
layout: post
title:  "Abrindo minha caixa de ferramentas"
date:   2016-12-11 11:24:00
categories: toolbox offtopic
---

Neste artigo irei compartilhar algumas ferramentas que uso para desenvolver software no dia a dia, vou falar desde sistema operacional chegando até o uso do CURL.

## Sistema Operacional

No trabalho e em casa utilizo bastante o **macOS**, apesar que ultimamente em casa estou utilizando mais o **Arch Linux**. As vantagens de se utilizar o **macOS** é que você terá um sistema operacional estável, com todas as vantagens da plataforma **UNIX**, e o **homebrew** como um ótimo gerenciador de pacotes. A grande desvantagem é que o equipamento é caro, ficando difícil sempre manter o **hardware** atualizado por conta do preço. Por isso em relação ao SO, recomendo fortemente o **Arch Linux** pelos motivos:

* Você não ficará preso ao **hardware** proprietário, poderá comprar qualquer pc ou notebook;
* Possui um excelente gerenciador de pacotes, o **pacman**, nele você nunca terá problemas com versões desatualizadas de pacotes;
* Instalação limpa, pois quando você instala o Arch, ele vem somente com o console, então caberá a você instalar o gerenciador de janelas, e tudo mais, assim será instalado somente o que você realmente necessita;
* A sensação legal de ter sempre rodando a versão atual do *kernel* do **Linux**.

## Desktop do Linux

Sou fã e usuário do **Gnome**, ele tem uma interface bastante *clean*, sobrando bastante espaço para o conteúdo que tem ser exibido nas janelas, além de já vir com o **Gnome Terminal**. Mas um outro desktop que acho bacana é o **XFCE**, bem limpo e rápido também.

## Editor de Texto

Como editor de texto uso unicamente o **VIM**, dominando seus **comandos** ele me oferece um grande ganho de produtividade. Apesar de eu ter testado um pouco o **Emacs**, que possui uma gama de **plugins**, eu fico no lema **menos é mais**, com **terminal** + **vim** faço qualquer coisa, risos. Antes do **VIM**, eu utilizava bastante o **Sublime Text**, mas acredito que atualmente existem melhores opções neste mesmo segmento, como o **Atom**. Se você está querendo começar no **VIM**, vou dar as seguintes dicas:

* Primeiro aprenda a utilizar os comandos básicos, pesquisando no google, você achará muita coisa;
* Crie seu próprio dotfile *.vimrc*, utilize o *Vundle* para gerenciar *plugins*;
* Seja persistente, a ideia do editor é outra, mas no final irá valer a pena.

Essa [tabela periódica do vim](http://www.viemu.com/vi-vim-cheat-sheet.gif) vai te ajudar com os comandos, utilizei bastante ela quando iniciei no **VIM**.

Recomendo dar uma olhada no meu [.vimrc](https://github.com/sergioaugrod/dotfiles/blob/master/.vimrc), nele você encontrará vários *plugins* quase indispensáveis e já vai estar configurado com um visual legal.

## Terminal

Utilizo e recomendo o **zsh**, ele fornece algumas facilidades como *auto complete*, *tab complete*... Eu utilizo um *framework* que adiciona o *zsh* junto com alguns *plugins* e facilidades o https://github.com/robbyrussell/oh-my-zsh, ele é bem fácil, e já deixa seu terminal com um visual bacana. No **macOS** eu utilizo o **iTerm** como *player* do terminal, ele possui a excelente *feature* de *splits* na janela. No **Linux** e as vezes no macOS, eu utilizo o **tmux**, para também fazer *splits*, criar várias *windows*, compartilhar sessões, entre outras coisas.

## Browser

Utilizo o **Chrome**, pois ele possui uma excelente ferramente de console, o **Developer Tools**, que facilita bastante para debugar e testar o **JavaScript**.

## CURL

Quem nunca usou ferramentas como **Postman** para interagir com APIS? Ou instalou a extensão **JSON Viewer** pra conseguir ver a resposta de um *GET* bonitinho no *browser*? As vezes não é necessário essas coisas, tudo que você precisa é o **CURL** com o **JSONPP**:

Quero listar os usuários da minha API:

{% highlight shell_session %}
$ curl http://localhost:3000/users | json_pp
{% endhighlight %}

Quero criar um novo produto na minha API:

{% highlight shell_session %}
$ curl -X POST http://localhost:3000/products -H "Content-Type: application/json" -d '{"description":"Product", "price":"10.55"}'
{% endhighlight %}

Simples não? E você faz tudo isso sem sair do seu terminal.

## Concluindo

Tentei mostrar algumas ferramentas que utilizo bastante no meu dia a dia, devo ter esquecido várias, mas acho que lembrei das importantes. Fui bem sucinto nesse *artigo*, mas já estou programando fazer vários *posts* sobre cada uma dessas ferramentas, dando maior detalhes da instalação, uso e configuração. Espero que tenham gostado e até mais!

## Referências e Links

* <https://wiki.archlinux.org/index.php/installation_guide>
* <https://github.com/sergioaugrod/dotfiles>
* <http://www.viemu.com/vi-vim-cheat-sheet.gif>
* <https://scotch.io/tutorials/getting-started-with-vim-an-interactive-guide>
* <https://tmux.github.io>
* <https://www.iterm2.com>
* <https://developer.chrome.com/devtools>
* <https://github.com/curl/curl>
