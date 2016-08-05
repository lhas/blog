---
title: Escrevendo componentes com AngularJS no padrão do Google
date: 2016-08-02 11:28:49
tags: ["components", "angularjs", "google", "patterns"]
cover: cover.jpg
share_cover: /2016/08/02/Escrevendo-componentes-com-AngularJS-no-padrao-do-Google/cover.jpg

---

Depois de ter passado por um bloqueio criativo de duas semanas, resolvi escrever sobre um assunto que perpetuou minha cabeça nos últimos dias.

Foi complexo pensar em alguma informação para compartilhar no blog sem parecer repetitivo, já que alguns autores gringos já abordaram este assunto [como aqui](http://teropa.info/blog/2015/10/18/refactoring-angular-apps-to-components.html).

# Introdução

Independente do fato de se discutir se o [Angular 2](https://angular.io/) vai vingar ou não, 
é interessante que devs de Angular1 aprendam a organizar suas aplicações sob a perspectiva 
de componentes, pois isso beneficia todo o time envolvido como veremos mais abaixo.

# O que é este "Component Style"?

Provavelmente a aplicação que você está mexendo agora segue numa estrutura semelhante a isto:
```
/app/
- controllers/
- services/
- factories/
- directives/
- models/
```
Se esta suposição estiver correta, sinto lhe informar mas...

![You're doing it wrong](youre-doing-it-wrong.jpg)

O problema desta arquitetura é que conforme a aplicação vai se formando, a manutenção do mesmo se torna complexa e trabalhosa.

Um exemplo prático: digamos que queremos refatorar algo na página Matrix. Para isto vamos precisar modificar:

```
/app/controllers/MatrixController.js
/app/services/MatrixService.js
/app/directives/MatrixListDirective.js
/app/models/Matrix.js
```

Mesmo que seja uma modificação simples, vamos precisar mexer em 4 pastas diferentes para conseguirmos mexer em uma única página.

Isso sem contar se além disto, você precisar modificar algo na camada de autenticação ou nos testes, coloque mais 4 arquivos diferentes em pastas diferentes e o caos está formado.

Foi pensando nisso que a equipe do Google resolveu expor uma arquitetura diferente para nós desenvolvedores organizarmos melhor nosso código e podermos fazer manutenções em tempo recorde.

A arquitetura de componentes não é algo novo e já é usado inclusive por alguns frameworks. É mais comum você ver este approach em stacks NodeJS.

No caso do nosso exemplo acima, no padrão de componentes seria assim:

```
/app/
- matrix/
- - matrix.controller.js
- - matrix.controller.spec.js (se estiver mexendo com testes)
- - matrix.service.js
- - matrix.service.spec.js (se estiver mexendo com testes)
- - matrix.html
- - matrix.scss
```

No caso, temos todo o CSS, toda a camada de testes, toda a camada de visualização, dentros de uma única pasta.

Imagine que vamos fazer uma nova aplicação chamada MatrixRevolution, e nisto vamos reaproveitar a nossa amada página Matrix.

Se fôssemos manter o nosso primeiro exemplo como padrão, seria uma loucura no Ctrl+C/Ctrl+V, pois você precisaria acessar a pasta de cada camada.

No padrão dos componentes, você só copia a pasta matrix/ e pronto. Todo CSS, HTML, Testes, JS em um só lugar. Simples, não?

Mas não para por aí.

# Como usar componentes

Você deve estar se perguntando. É mais fácil do que parece. 

Você pode inclusive ir adaptando aos poucos a sua aplicação para que
esta mudança seja progressiva e não-agressiva ao resto da equipe de desenvolvimento.

Vamos ao exemplo prático:

Precisamos que você faça um **fórum** dentra nossa atual aplicação.

Crie dentro da pasta da aplicação uma pasta chamada "forum".

**Dentro do fórum, temos:**
- Listagem de seções **(board.list)**
- Dentro da seção, temos a listagem de tópicos **(board)**
- Dentro do tópico, temos a listagem de respostas **(topic)**

Nossa estrutura de pastas será:
```
app/
- forum/
- - board/
- - - list/
- - topic/
```

Neste tutorial, vou explicar somente a parte do tópico.

Quem sabe depois escrevo o resto dos outros componentes.

Para o tópico vamos precisar:

## Controller

Os arquivos serão:
```
app/forum/topic/topic.controller.js
app/forum/topic/topic.controller.spec.js
```

Dentro do controller, vou colocar aqui um exemplo que já está nos padrões de código do [John Papa, o Angular 1 Style Guide](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md)

```js
/* topic.controller.js */
(function(){
    'use strict';

    angular
        .module('app.forum.topic', [])
        .component('matrixTopic', {
            templateUrl: 'app/forum/topic/topic.html',
            controller: TopicController,
            controllerAs: 'ctrl'
        })

        TopicController.$inject = ['$stateParams', 'TopicService'];

        function TopicService($stateParams, TopicService) {
            var vm = this;
            var topic = undefined;

            vm.topic = topic;
            
            activate();

            function activate() {
                if(vm.topic == undefined) {
                    TopicService.getOne($stateParams.id).then(success, error);

                    function success(result) {
                        vm.topic = result.data.topic;
                    }

                    function error(error) {
                        console.log('Error ocurred: ' + error);
                    }
                }
            }
        }
})();
```

Nosso controller está pronto!

Recuperando o tópico com o TopicService e atualizando nosso ViewModel (daí o vm) 
com o retorno da API.

## Service
Os arquivos serão:
```
app/forum/topic/topic.service.js
app/forum/topic/topic.service.spec.js
```

Esta é a camada que vai entrar em contato com a API. Ela é usada no nosso controller acima.

Nosso código nos padrões do John Papa:

```js
/* topic.service.js */
(function() {
  'use strict';

  angular
    .module('app.forum.topic')
    .service('TopicService', TopicService);

  TopicService.$inject = ['$http', 'API_URL', '$translate'];

  // API_URL no caso seria uma constante
  function TopicService($http, API_URL) {

    var service = {
      getAll: getAll,
      getFeatured: getFeatured,
      getOne: getOne
    };

    return service;
    
    /** Get all topics from the API. */
    function getAll() {
      return $http.get(API_URL + 'topic', {
        params: {
          language: 'br'
        }
      });
    }
    
    /** Get featured topics from the API. */
    function getFeatured(limit) {
      return $http.get(API_URL + 'topic/featured');
    }

    /** Get one single topic from the API. */
    function getOne(id) {
      return $http.get(API_URL + 'topic/' + id, {
        params: {
          language: 'br'
        }
      });
    }

  }

})();
```

## View

O arquivo será:
```
app/forum/topic/topic.html
```

A parte mais simples de todas. É o que vai renderizar dentro do nosso componente.

```html
<!-- topic.html -->
<div class="topic well">
    <h1>{{ctrl.topic.title}}</h1>

    <p>{{ctrl.topic.content}}</p>

    <h2>Respostas</h2>

    <ul class="list-group">
        <li class="list-group-item" ng-repeat="reply in ctrl.topic.replies">
            {{reply.author}} disse:

            <p>{{reply.content}}</p>

            <em>{{reply.created_at | date 'd/m/Y'}}</em>
        </li>
    </ul>
</div>

```

## SCSS

O arquivo será:
```
app/forum/topic/topic.scss
```

A segunda parte mais simples aqui.

```scss
    .topic {
        background: #F00;
        color: #FFF;
    }
```

É importante na view e no CSS você usar classes ao invés de IDs, pois o conceito por trás dos componentes é que eles sejam reutilizáveis o máximo possível.

Não esqueça de incorporar o topic.scss no seu main.scss com um @import da vida:

```scss
    @import "app/forum/topic/topic.scss";
```


Agora só chamar no seu index.html:

```html
    <script src="app/forum/topic/topic.controller.js"></script>
    <script src="app/forum/topic/topic.service.js"></script>
```

E no seu module.js incluir o seu novo módulo como dependência:

```js
(function(){

    angular
        .module('app', ['app.forum.topic']);
})();
```

Agora para usar o componente, digamos no seu routes.js usando o UI Router:

```js
angular
  .module('app')
  .config(routesConfig);

routesConfig.$inject = ['$stateProvider', '$urlRouterProvider', '$locationProvider'];

function routesConfig($stateProvider, $urlRouterProvider, $locationProvider) {
  $locationProvider.html5Mode(true).hashPrefix('!');
  $urlRouterProvider.otherwise('/');

  $stateProvider
    .state('home', {
      url: '/',
      template: '<matrix-home></matrix-home>'
    })
    .state('topic', {
      url: '/topic/{id}',
      template: '<matrix-topic></matrix-topic>'
    });
}
```

Pronto! :D

Agora toda vez que você quiser mostrar um tópico na aplicação, só colocar em qualquer HTML:

```html
<matrix-topic></matrix-topic>
```

Espero que gostem. Quero aprofundar mais este assunto aqui depois.