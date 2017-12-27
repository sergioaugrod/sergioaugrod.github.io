---
title: Criando um jogo multiplayer com Elixir, Phoenix Framework e Phaser
layout: post
date: '2017-12-27 01:00:00'
disqus: 'true'
categories: elixir games
---

Há pouco mais de 1 mês criei um projeto no [Github](https://github.com/sergioaugrod/uai_shot) chamado **UaiShot**, um jogo *multiplayer* de naves, que o objetivo é acertar as outras naves para ficar melhor colocado no *ranking*. Resolvi criar este jogo para explorar um pouco das vantagens dos *channels* do *Phoenix Framework*, e fazer algo com o *Phaser*. O resultado final ficou interessante, então resolvi compartilhar um pouco do processo de desenvolvimento com vocês.

## Primeiros passos

Para criar o jogo, resolvi utilizar o tradicional modelo de sistema distribuído **Cliente/Servidor.** Sendo assim, o servidor será o ponto central de comunicação, e tem como objetivo armazenar o estado e informar os clientes sobre atualizações nos dados. No caso específico do **UaiShot**, o estado será:

* Jogadores online;
* Posições das balas atiradas pelas naves do jogo;
* *Ranking*.

Após definir o modelo, pensei nas ações do jogo, já separando as responsabilidades do cliente e do servidor. Abaixo posso citar algumas delas:

### Ações do cliente para o servidor:

* **NEW PLAYER**: Um novo jogador entrou no jogo;
* **MOVE PLAYER**: Algum jogador se moveu com a nave;
* **SHOOT BULLET**: Algum jogador atirou com a sua nave.

### Ações do servidor para os clientes:

* **UPDATE PLAYERS**: Um novo jogador entrou no jogo ou se moveu;
* **UPDATE RANKING**: O *ranking* foi atualizado;
* **UPDATE BULLETS**: Algum jogador atirou com sua nave ou alguma bala está se movendo;
* **HIT PLAYER**: Algum jogador acertou outra nave.

Para facilitar o entendimento das ações, visualize o diagrama abaixo:

![Ações do Jogo](/assets/images/game_actions.png)

*Push é a ação de um cliente para o servidor. Broadcast é a ação do servidor para todos os clientes conectados.*

## Tecnologias

Agora que já falamos das ações do jogo, vamos falar um pouco sobre as tecnologias utilizadas no desenvolvimento.

### Elixir

**Elixir** é uma linguagem de programação funcional, que é executada na máquina virtual do *Erlang*, e se destaca quando o assunto é criação de sistemas com alto nível de concorrência. Podemos citar algumas das suas principais características:

* Paradigma Funcional;
* Escalabilidade por meio de processos leves;
* Comunicação entre processos é feita por mensagens;
* Metaprogramação;
* Macros;
* Extensível e fácil na criação de DSLs;
* Polimorfismo via protocolos;
* Tolerância a falhas.

A *syntax* da linguagem é muito elegante, bem parecida com a do *Ruby*, e conta com todas as vantagens do *Erlang*, que foi projetado na década de 80 para lidar com desafios de sistemas para telecomunicações. Atualmente estou trabalhando com sistemas em produção rodando em *Elixir*, e posso destacar que o ferramental para o desenvolvimento é muito bom, ferramentas como o **Mix** (builds, taks e compilação) e o **ExUnit** (testes de unidade) facilitam bastante o desenvolvimento.

### Phoenix Framework

**Phoenix** é um *framework* web para a linguagem *Elixir*. Sua experiência de uso é bem similar ao framework *Rails*, com suas particularidades devido ao paradigma da linguagem *Elixir*. Um dos seus principais *cases* é a sua escalabilidade quando o assunto é *web sockets*. Atualmente é uma das melhores opções quando o requisito de sua aplicação é ser *realtime*, além da escalabilidade citada anteriormente, a facilidade do uso também é um grande destaque.

### Phaser

O **Phaser** é um *framework* *Javascript*, que normalmente é a primeira opção quando se pesquisa sobre desenvolvimento de jogos no navegador. Posso destacar sua documentação com diversos exemplos de jogos,  e a sua facilidade de uso.

## Desenvolvimento

Vamos começar falando um pouco sobre o *front-end* do jogo, explicando como funciona o *bootstrap* de uma aplicação *Phaser*, e por final como é a comunicação via *websockets*. Depois vamos para o *back-end*, para explicar como é armazenado e atualizado o estado do jogo. Nesta seção irei demonstrar partes do código do **UaiShot**, para visualizar o código fonte completo, visite o [repositório](https://github.com/sergioaugrod/uai_shot) do projeto. Abaixo um diagrama contendo a arquitetura do jogo:

![Arquitetura do Jogo](/assets/images/game_architecture.png)

### Cliente

No **UaiShot** o *client* não é uma aplicação separada, ele fica dentro da estrutura do **Phoenix Framework**. Para iniciar um projeto *Phaser* dentro do *Phoenix*, você deverá executar dois passos:

* Inserir o arquivo minificado do *Phaser* no diretório `assets/vendor/js` do projeto;
* Adicionar a *tag* `<div id="phaser"></div>` no template principal da sua aplicação, no meu caso em `lib/uai_shot_web/templates/page/index.html.ex`.

#### app.js

O *front-end* do nosso jogo contém 3 arquivos, todos localizados no diretório `assets/js`. Vamos começar pelo `app.js`, que é o arquivo onde instanciamos o *Phaser*:

{% highlight javascript %}
import {Game} from "./game";

let nickname = prompt("Hello! What's your name?");
let engine = new Phaser.Game(800, 600, Phaser.CANVAS, "phaser", { preload: preload, create: create, update: update });
let game = new Game(engine, nickname);
game.start();

function preload() {
  game.preload(this);
}

function create() {
  game.create(this);
}

function update() {
  game.update(this);
}
{% endhighlight %}

É no `app.js` onde perguntamos qual vai ser o `nickname` do jogador, e dizemos ao *Phaser* qual o tamanho da tela. O importante deste arquivo é a implementação dos *callbacks*  do *framework* *Phaser*:

* **preload**: É onde se deve definir os *assets* da nossa aplicação, como por exemplo: nave, *background* e a bala;
* **create**:  Responsável por criar a nave do jogador e definir os comandos;
* **update**: É executada quando o jogador realiza ações no jogo.

#### game.js

Toda a implementação dessas funções está localizada no arquivo `game.js`:

{% highlight javascript %}
preload(state) {
  this.engine.load.image("bullet", "images/bullet.png");
  this.engine.load.image("ship", "images/ship.png");
  this.engine.load.image("space", "images/space.png");
}

create(state) {
  this.engine.add.tileSprite(0, 0, 800, 600, "space");
  this._setTexts();
  this._setKeyboard(state);
  this._createPlayer();
}

update(state) {
  this.player.update(this.engine, this.cursors, this.shootButton, this.channel);
  this._updateAlpha();
}
{% endhighlight %}

O arquivo `game.js` é responsável por:

* Criar as naves;
* Atualizar *ranking* e número de jogadores online;
* Receber eventos do servidor como  **UPDATE PLAYERS**,  **UPDATE BULLETS**, **UPDATE RANKING** e **HIT PLAYER**;
* Enviar eventos para o servidor como **NEW PLAYER**.

#### player.js

O arquivo `player.js` é responsável por:

* Definir os controles do jogador;
* Enviar eventos para o *servidor* como **SHOOT BULLET** e **MOVE PLAYER**.

#### Comunicação com o servidor via websockets

O **Phoenix Framework** tem os [Channels](https://hexdocs.pm/phoenix/channels.html), que são estruturas parecidas com *Controllers*, porém para comunicação via *websockets*. Basicamente um *channel* tem um conjunto de *callbacks* para eventos. Sendo assim, se o *client* enviar um evento *move_player* para o *GameChannel*, então deverá ter um evento *move_player* definido no *GameChannel*.

No arquivo `game.js` é onde fazemos a conexão com *socket*  e os *channels* do **Phoenix Framework**:

{% highlight javascript %}
_connectToLobby() {
  let socket = new Socket("/socket", { params: { nickname: this.nickname } });
  socket.connect();
  let channel = socket.channel("game:lobby", {});
  channel
    .join()
    .receive("ok", payload => this.playerId = payload.player_id);
  this.channel = channel;
}
{% endhighlight %}

O código acima se conecta com o *socket* e envia o nickname. Após isso, ele se conecta com o channel *game:lobby*. 

A cada movimento do jogador é enviado a posição *x* e *y* da nave, e também seu ângulo de rotação.  Este evento é recebido no servidor, e repassado para todos os outros jogadores. Abaixo um exemplo do **envio do evento** *move_player*:

{% highlight javascript %}
_movePlayer(channel) {
  channel.push("move_player", { x: this.sprite.x, y: this.sprite.y, rotation: this.sprite.rotation });
}
{% endhighlight %}

Quando uma nave é atingida por uma bala, recebemos o evento *hit_player* no *client*:

{% highlight javascript %}
_hitPlayer() {
  this.channel.on("hit_player", payload => {
    if(this.playerId != payload.player_id) {
      this.players[payload.player_id].alpha = 0;
    } else {
      this.player.sprite.alpha = 0;
    }
  });
}
{% endhighlight %}

### Servidor

O servidor é responsável por receber/enviar os eventos do jogador e gerenciar o estado do jogo. Nele também está presente a lógica para identificar se alguma nave foi atingida por uma bala. Caso uma nave seja atingida por uma bala, o jogador perderá 10 pontos, caso ele atinja uma outra nave, ele ganhará 10 pontos. Posso dividir o nosso *back-end* em 3 partes: `Stores`, `GameChannel` e `GameServer`.

#### Stores

Como armazenamos o estado das coisas em uma linguagem funcional? Não é simplesmente criar um `array` dentro do módulo, e inserir os elementos. No **Elixir** os processos possuem um estado, então para armazenar nossos dados, devemos simplesmente criar um processo. O [Agent](https://hexdocs.pm/elixir/Agent.html) no Elixir, é um tipo de processo especializado em gerenciar um estado, ele conta com uma *interface* para atualizar e retornar um estado. Por exemplo:

{% highlight elixir %}
defmodule UaiShot.Store.Player do
  use Agent

  def start_link do
    Agent.start_link(fn -> %{} end, name: __MODULE__)
  end

  def put(player) do
    Agent.update(__MODULE__, &Map.put(&1, player.id, player))
  end

  def get(player_id) do
    Agent.get(__MODULE__, &Map.get(&1, player_id, default_attrs(player_id)))
  end
end
{% endhighlight %}

No código acima, temos o `start_link` responsável por iniciar nosso `Agent`, e mais duas funções. A função `put` é uma interface para a função `update` do módulo `Agent`, que basicamente aplica uma função sobre o seu estado atual, neste caso adicionando mais um elemento no `Map`. Já a função `get` retorna o valor do `Map` conforme o identificador do jogador.

No **UaiShot** possuímos 3 `agents`. Eles são responsáveis por armazenar o estado dos jogadores, *ranking* e balas.

#### GameChannel

O módulo **GameChannel** é responsável por 3 coisas:

* Receber evento de novos jogadores;
* Receber evento da movimentação dos jogadores;
* Receber evento do disparo de balas pelas naves.

Ao receber um novo jogador, atualizamos a lista de jogadores e informamos a todos os outros jogadores sobre o estado atual da lista:

{% highlight elixir %}
def handle_in("new_player", state, socket) do
  state = format_state(state)
  nickname = socket.assigns.nickname
  player_id = socket.assigns.player_id

  state
  |> Map.put(:id, player_id)
  |> Map.put(:nickname, nickname)
  |> Player.put()

  Ranking.put(%{player_id: socket.assigns.player_id,
                nickname: nickname,
                value: 0})

  broadcast(socket, "update_players", %{players: Player.all})
  broadcast(socket, "update_bullets", %{bullets: Bullet.all})
  broadcast(socket, "update_ranking", %{ranking: Ranking.all})

  {:noreply, socket}
end
{% endhighlight %}

Ao receber o evento da movimentação de um jogador, atualizamos os dados do jogador na lista de jogadores e informamos a todos os outros jogadores sobre o estado atual da lista:

{% highlight elixir %}
def handle_in("move_player", state, socket) do
  state
  |> format_state()
  |> Map.put(:id, socket.assigns.player_id)
  |> Map.put(:nickname, socket.assigns.nickname)
  |> Player.put()

  broadcast(socket, "update_players", %{players: Player.all})

  {:noreply, socket}
end
{% endhighlight %}

Ao receber o evento do disparo de bala por uma nave, atualizamos a lista de balas e informamos a todos os jogadores sobre o estado atual da lista:

{% highlight elixir %}
def handle_in("shoot_bullet", state, socket) do
  state
  |> format_state()
  |> Map.put(:player_id, socket.assigns.player_id)
  |> Bullet.push()

  broadcast(socket, "update_bullets", %{bullets: Bullet.all})

  {:noreply, socket}
end
{% endhighlight %}


#### GameServer

Para finalizar a seção de desenvolvimento, temos o `GameServer`, que é responsável por executar as *engines* do jogo. Atualmente o **UaiShot** possui somente a *engine* de batalha. A `Battle Engine` basicamente move as balas do jogo, verifica se uma bala acertou uma nave e também atualiza o *ranking*. O `GameServer` dispara um *broadcast* para o *client* ao realizar cada uma dessas ações. Abaixo o código fonte deste módulo:

{% highlight elixir %}
defmodule UaiShot.Engine.Battle do
  alias UaiShot.Store.{Bullet, Player, Ranking}
  alias UaiShotWeb.Endpoint

  @game_width 800
  @game_height 600

  def run do
    bullets = Bullet.all()
    |> Enum.map(&move_bullet(&1))
    |> Enum.reject(&bullet_is_far?(&1))

    Bullet.reset(bullets)
    Endpoint.broadcast("game:lobby", "update_bullets", %{bullets: bullets})
  end

  defp move_bullet(bullet) do
    bullet
    |> hited_players()
    |> Enum.each(&process_hit(bullet, &1))

    bullet
    |> Map.put(:x, bullet.x + bullet.speed_x)
    |> Map.put(:y, bullet.y + bullet.speed_y)
  end

  defp process_hit(bullet, player) do
    update_ranking(bullet.player_id, 10)
    update_ranking(player.id, -10)

    Endpoint.broadcast("game:lobby", "update_ranking", %{ranking: Ranking.all()})
    Endpoint.broadcast("game:lobby", "hit_player", %{player_id: player.id})
  end

  defp update_ranking(player_id, value) do
    player_id
    |> Ranking.get()
    |> Map.update!(:value, &(&1 + value))
    |> Ranking.put()
  end

  defp bullet_is_far?(bullet) do
    bullet.x < -10 || bullet.x > @game_width
    || bullet.y < -10 || bullet.y > @game_height
  end

  defp hited_players(bullet) do
    Player.all()
    |> Enum.filter(&(bullet.player_id != &1.id))
    |> Enum.filter(fn player ->
      dx = player.x - bullet.x
      dy = player.y - bullet.y
      :math.sqrt(dx * dx + dy * dy) < 30
    end)
  end
end
{% endhighlight %}

## Conclusão

Neste artigo tentei demonstrar um pouco sobre como é desenvolver um jogo *multiplayer* bem simples. Posso resumir que a construção deste jogo, foi basicamente controlar o estado dos jogadores e verificar quando uma nave é atingida. A cada ação e movimentação de um jogador/bala sempre devemos avisar aos demais. No desenvolvimento do **UaiShot** consegui validar o quanto *Elixir* e o *Phoenix* são excelentes ferramentas para construir aplicações realtime, e também, quão fácil é desenvolver um jogo com o *Phaser*. 

## Referências

* https://github.com/sergioaugrod/uai_shot
* https://elixir-lang.org
* https://hexdocs.pm/elixir/Agent.html
* http://phoenixframework.org
* https://hexdocs.pm/phoenix/channels.html
* https://phaser.io
* https://code.tutsplus.com/tutorials/create-a-multiplayer-pirate-shooter-game-in-your-browser--cms-23311
* https://www.youtube.com/watch?v=I5L9_cXwBcU&t=1069s