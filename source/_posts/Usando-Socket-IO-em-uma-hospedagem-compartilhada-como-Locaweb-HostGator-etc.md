---
title: >-
  Usando Socket.IO em uma hospedagem compartilhada (como Locaweb, HostGator,
  etc)
date: 2016-07-14 02:10:28
tags: ["node.js", "socket.io", "hospedagem compartilhada", "vps", "digitalocean"]
cover: cover.jpg
share_cover: /2016/07/14/Usando-Socket-IO-em-uma-hospedagem-compartilhada-como-Locaweb-HostGator-etc/cover.jpg
---

Hoje vou ensinar como trabalhar com o Socket.IO para transformar sua aplicação web em tempo real.

# O que é Socket.IO?

Segundo a descrição no [site](http://socket.io/) deles, "THE FASTEST AND MOST RELIABLE REAL-TIME ENGINE".

Ele permite que você crie aplicações em tempo real de forma bi-direcional (tanto o cliente quanto o servidor podem se comunicar), orientado a eventos.

Ela funciona em qualquer dispositivo, browser, plataforma e você não deve ter problemas.

Sites como o [Trello](https://trello.com) usam e abusam deste recurso para fazer o site funcionar em tempo real, seja com as notificações ou as interações nos cards.

O foco deste tutorial é somente de instalá-lo e usá-lo em um site básico que possa estar hospedado em uma hospedagem compartilhada. Outros tutoriais virão sobre como usá-lo o Socket.IO melhor.

# O que vou precisar?

- Terminal
- Node.JS
- Conhecimento em JavaScript
- Um VPS (e acesso SSH a ele)

# Começando...

1) Acesse o servidor que vai hospedar a aplicação via SSH.
```bash
ssh root@0e1dev.com
```
2) Instale o Node.JS (não abordarei isto aqui, você pode aprender sobre [aqui](https://www.digitalocean.com/community/tutorials/como-instalar-o-node-js-em-um-servidor-ubuntu-14-04-pt)).
3) Instale o PM2:

```bash
npm install pm2 -g
```

4) Vamos montar nosso script? Bora:

Crie uma pasta para nosso app, e dentro um arquivo chamado server.js.
```
/app
- server.js
```

Primeiro precisamos instalar nossas dependências (Express e o Socket.IO). O Express é necessário pois para o Socket.IO rodar ele precisa de um processo HTTP já rodando em algum lugar. Para instalar rode este comando na pasta do app:
```bash
npm install --save-dev express socket.io
```

Agora, vamos editar o server.js.

No começo do nosso script, vamos importar as dependencias que instalamos:
```javascript
var app = require('express')();
var http = require('http').Server(app);
var io = require('socket.io')(http);
```

Caso você queira criar uma rota para testar o funcionamento do Express na nossa app, adicione logo depois:
```javascript
app.get('/', function(req, res){
  res.send('<h1>Hello world</h1>');
});
```

Como caso de uso, vamos fazer um counter de usuários acessando nosso site. Para isto, vamos declarar uma variável que vai armazenar este total. Depois, vamos manipular o evento "connection" e "disconnect" disponíveis dentro do socket. Quando ocorrer uma nova "connection", vamos acrescentar esta variável total. Quando ocorrer um "disconnect", vamos reduzir essa variável.

Sim, o Socket.IO permite que nós detectamos quando um usuário "sai" do nosso site. Isso é muito bom, levando em conta que a um tempo atrás não era possível fazer esse tipo de detecção client-side apenas.

Chega de papo e vamos pro código:

```javascript
// Armazena o total de usuários
var totalUsers = 0;

// Evento - Ao ocorrer uma conexão (um novo socket disponível)
io.on('connection', function(socket){

  // Acrescentar total
  totalUsers++;

  // Vamos emitir um evento chamado "refresh users". Ele será usado depois
  io.emit('refresh users', totalUsers);

  // Caso ocorra um disconnect, reduzimos o total de usuários e emitimos o evento "refresh users" novamente, com o novo total.
  socket.on('disconnect', function() {
    totalUsers--;

    io.emit('refresh users', totalUsers);
  });

});
```

Feito isso, precisamos inicializar nosso servidor web. Continue escrevendo:

```javascript
http.listen(3000, function(){
  console.log('listening on *:3000');
});
```

Pronto! Já temos um script que configura o ambiente do Socket.IO através da porta 3000. Basta rodá-lo seja na sua máquina quanto no seu servidor/VPS que estará funcionando.

# Como colocar ele pra funcionar no servidor?

No começo sugeri que instalasse o PM2. Ele é um gerenciador de aplicações Node.JS. Perfeito para ambientes de produção.

O mais bacana de usar o PM2 para este tipo de situação é usar em paralelo a ferramenta deles chamada [Keymetrics](https://keymetrics.io/). É uma ferramenta muito bacana e eu sugiro que você a experimente **AGORA**!

Você pode monitorar em TEMPO REAL (UAU!) suas aplicações escritas em Node. Isso significa que se por exemplo, sua aplicação lançar uma exceção, eles te notificarão por e-mail que uma exceção ocorreu na aplicação X na hora Y pelo usuário Z. É muito foda, apenas use.

Enfim, após instalar o PM2 você terá alguns comandos disponíveis:

```bash
// Mostra todas as aplicações startadas usando o PM2
pm2 list
// Inicia uma nova aplicação com o PM2, basta rodá-lo na pasta do script
pm2 start server.js
//  Mostra informações sobre memória e CPU sendo consumidas pelas aplicações
pm2 monit
// Reinicia o NOME_APP
pm2 restart [NOME_APP]
```

Basta subir o script para o servidor, acessar a pasta onde o script está localizado por SSH e rodar:

```bash
pm2 start server.js
```

Agora caso você abra http://seu.servidor:3000/

Você vai ver o Hello World.

Para manipular o Socket.IO no seu site agora, basta importá-lo usando a URL do seu servidor:

```html
  <script src="http://0e1dev.com:3000/socket.io/socket.io.js"></script>
  <script>
    var socket = io();

    socket.on('refresh users', function(data) {
      console.log("Total de usuários logados: " + data);
    });
  </script>
```

# E se eu não tiver acesso a um VPS?

Você pode recorrer a alternativas como o [Heroku](https://www.heroku.com/) ou [OpenShift](https://www.openshift.com/) (mantido pela Red Hat). Ambos contém planos gratuitos onde você hospedar esta aplicação em Node.JS tranquilamente.

É apenas uma alternativa usar esses PaaS e **não recomendo** esta solução para um projeto que vá realmente para o ambiente de **produção**.

# Quais são os benefícios de utilizar este recurso?

Você pode incrementar qualquer site com recursos bacanas. No caso de uma rede social por exemplo, você pode desenvolver chats em tempo real, notificações (quem visitou seu perfil, comentou, curtiu algo), jogos(por que não?), etc.

Sites que usam deste tipo de recurso costumam ser mais bem vistos em comparação aos seus concorrentes. Além de é claro, contribuir para uma melhor navegabilidade e experiência pro usuário final.

# Quais são as limitações dele?

Como qualquer aplicação Node.JS ele precisará de um ambiente que permita este tipo de aplicação. Por embaixo dos panos, ele trabalha com o WebSocket. Outras ferramentas fazem essa mesma integração com o WebSocket, como é o caso do [SockJS](https://github.com/sockjs/sockjs-client).

O Trello em 2012 teve um problema com ter mais de 10 mil conexões simultâneas. Eles fizeram um patch para resolver isso na época, mas hoje em dia o problema aparentemente foi solucionado por parte do Socket.IO. [Saiba mais sobre o caso aqui](http://blog.fogcreek.com/the-trello-tech-stack/).

Pela internet você encontra vários artigos de pessoas relatando terem conseguido até UM MILHÃO de conexões simultâneas. Mas são situações bem específicas. Neste caso o ideal é escalar.

# Isso é uma gambiarra?

Sim. >:-)

# FIM!

Por enquanto é isso.

Espero que seja produtivo para alguém a leitura disto.

No meu caso, estou desenvolvendo uma aplicação que a princípio ficará em uma hospedagem compartilhada mas que nós na 0e1 precisamos entregar features em tempo real nos próximos sprints.

Nós rodamos o Socket.IO no nosso servidor a penas fazemos a referência a ele no nosso site final. E tudo funciona perfeitamente bem! ;-)