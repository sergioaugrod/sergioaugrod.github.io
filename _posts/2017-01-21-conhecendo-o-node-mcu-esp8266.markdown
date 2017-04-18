---
layout: post
title:  "Conhecendo o Node MCU ESP8266"
date:   2017-01-21 23:12:00
categories: iot
---

O **Node MCU** é uma plaquinha bem interessante, construída em cima do **ESP8266**, ele já vem com *WiFi*, 10 pinos *GPIO* e um ótimo *firmware*, com a possibilidade da inclusão de diversas bibliotecas (MQTT, Web Sockets), e o melhor de tudo, programável pela excelente linguagem **Lua**. A ideia deste artigo é apresentar a placa, e ensinar desde subir um *firmware* customizado com alguns módulos, até interagir com a placa com código Lua.

#### Coisas legais do Node MCU

Começando pelo que podemos fazer de interessante com a placa:

* Executar servidor HTTP dentro da placa;
* Cliente HTTP;
* Interação com broker MQTT;
* Criação de Web Sockets;
* Interatividade com a linguagem Lua;
* Entrada/Saída digital.

#### Upload do Firmware

Agora que já sabemos um pouco da placa, vamos para o *firmware*. Provavelmente sua placa já virá com o *firmware*, mas é legal também saber fazer o upload de uma versão mais atualizada, além de incluir algumas bibliotecas extras.

Primeiramente vamos baixar o *firmware*, para isso recomendo o [Node MCU Custom Builds](https://nodemcu-build.com), lá você terá a versão atual e poderá adicionar algumas bibliotecas (é necessário preencher seu e-mail na ferramenta para que eles enviem o link do build customizado que você criou). Se der tudo certo, eles enviarão dois links, um da versão float e outro da versão integer(mais leve).

**Após baixar um dos links, iremos utilizar a ferramenta [esptool](https://github.com/espressif/esptool) para fazer o upload, primeiramente vamos instalar:**

{% highlight shell_session %}
$ pip install esptool
{% endhighlight %}

*É necessário ter o Python instalado em sua máquina.*

**Para utilizar é bem simples, devemos especificar o caminho do firmware baixado, a posição de memória inicial e a porta USB que o Node MCU está conectado:**

{% highlight shell_session %}
$ esptool.py --port /dev/cu.wchusbserial1420 write_flash 0x00000 nodemcu-master-float.bin
{% endhighlight %}

**Se tudo der certo, o *esptool* retornará uma mensagem mais ou menos assim:**

```
esptool.py v1.3
Connecting....
Auto-detected Flash size: 32m
Running Cesanta flasher stub...
Flash params set to 0x0040
Wrote 475136 bytes at 0x0 in 41.2 seconds (92.2 kbit/s)...
Leaving...
```

#### Interagindo com a placa

Para realizarmos essa interação com a plaquinha, iremos utilizar a excelente ferramenta [nodemcu-uploader](https://github.com/kmpm/nodemcu-uploader), para instalar:

{% highlight shell_session %}
$ pip install nodemcu-uploader
{% endhighlight %}

*Assim como o esptool, é necessário ter o Python instalado na máquina.*

**Vamos montar nosso *sketch* básico que irá acender o LED:**

![Sketch](/assets/images/nodemcu_sketch.jpg)

*O sketch consiste em LED, Resistor, Protoboard e dois jumpers.*

**Agora vamos criar nosso código Lua para acender o LED:**

{% highlight lua %}
gpio.mode(1, gpio.OUTPUT)
gpio.write(1, gpio.HIGH)
{% endhighlight %}

**Salve esse código com o nome *init.lua* e em seguida vamos subir para a placa:**

{% highlight shell_session %}
$ nodemcu-uploader --port /dev/cu.wchusbserial1420 upload init.lua
{% endhighlight %}

**Executando o código que subimos para a placa:**

{% highlight shell_session %}
$ nodemcu-uploader --port /dev/cu.wchusbserial1420 exec init.lua
{% endhighlight %}

**Executando comandos no interpretador Lua da placa:**

{% highlight shell_session %}
$ nodemcu-uploader --port /dev/cu.wchusbserial1420 terminal
{% endhighlight %}

*Aproveite para brincar um pouco com a linguagem Lua.*

**Para explorar mais opções do *nodemcu-uploader*:**

{% highlight shell_session %}
$ nodemcu-uploader -h
{% endhighlight %}

*Lembre-se de inserir o caminho da porta USB que está conectado o Node MCU.*

#### Exemplos de código:

**GPIO:**

{% highlight lua %}
gpio.mode(1, gpio.OUTPUT)
gpio.write(1, gpio.HIGH)
gpio.mode(1, gpio.INPUT)
print(gpio.read(1))
{% endhighlight %}

**Conectando ao WiFi:**

{% highlight lua %}
wifi.setmode(wifi.STATION)
wifi.sta.config("SSID","password")
{% endhighlight %}

**Servidor HTTP:**

{% highlight lua %}
srv=net.createServer(net.TCP)
srv:listen(80,function(conn)
    conn:on("receive",function(conn,payload)
    conn:send("<h1>www.sergioaugrod.com.br</h1>")
    end)
end)
{% endhighlight %}

#### Concluindo

Espero que este artigo possa ser útil para você que procura saber mais sobre essa excelente plaquinha, recomendo explorar mais exemplos de código na [documentação oficial](http://nodemcu.readthedocs.io/en/master/). Abraço!

#### Links

* <http://nodemcu.readthedocs.io/en/master>
* <https://github.com/sergioaugrod/node-mcu-samples>
* <https://nodemcu-build.com>
* <https://github.com/espressif/esptool>
* <https://github.com/kmpm/nodemcu-uploader>
